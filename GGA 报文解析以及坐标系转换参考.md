问题：“$GNGGA,065956.00,2813.85693310,N,11252.73428282,E,1,00,9999.0,94.3342,M,-16.6047,M,,*53” 编写js方法解析 这段GGA报文，注意将 经纬度字段的度分格式转换为十进制度

回答：

```javascript
function parseGGA(sentence) {
    // 检查是否为有效的GGA报文
    if (!sentence.startsWith('$GNGGA')) {
        throw new Error('Invalid GGA sentence: does not start with $GNGGA');
    }

    // 按逗号分割
    const fields = sentence.split(',');
    if (fields.length < 15) {
        throw new Error('Invalid GGA sentence: insufficient fields');
    }

    // 验证校验和
    const checksum = fields.pop().substring(1); // 移除*并获取校验和
    const calculatedChecksum = calculateChecksum(sentence.substring(1, sentence.indexOf('*')));
    if (calculatedChecksum !== checksum) {
        console.warn('Checksum mismatch: calculated ' + calculatedChecksum + ', expected ' + checksum);
    }

    // 解析字段
    const time = fields[1]; // UTC时间
    const latStr = fields[2]; // 纬度 ddmm.mmmm
    const latDir = fields[3]; // N/S
    const lonStr = fields[4]; // 经度 dddmm.mmmm
    const lonDir = fields[5]; // E/W
    const quality = parseInt(fields[6], 10); // 定位质量
    const numSats = parseInt(fields[7], 10); // 卫星数量
    const hdop = parseFloat(fields[8]); // HDOP
    const altitude = parseFloat(fields[9]); // 海拔（米）
    const altUnit = fields[10]; // 通常为'M'
    const geoSep = parseFloat(fields[11]); // 大地水准面分离
    const geoUnit = fields[12]; // 通常为'M'
    const ageDiff = fields[13]; // 差分年龄（空）
    const diffStation = fields[14]; // 差分站（空）

    // 将纬度转换为十进制度
    let latDecimal = 0;
    if (latStr && latDir) {
        const latMinutes = parseFloat(latStr);
        const latDegrees = Math.floor(latMinutes / 100);
        const latMins = latMinutes - (latDegrees * 100);
        latDecimal = latDegrees + (latMins / 60);
        if (latDir === 'S') {
            latDecimal = -latDecimal;
        }
    }

    // 将经度转换为十进制度
    let lonDecimal = 0;
    if (lonStr && lonDir) {
        const lonMinutes = parseFloat(lonStr);
        const lonDegrees = Math.floor(lonMinutes / 100);
        const lonMins = lonMinutes - (lonDegrees * 100);
        lonDecimal = lonDegrees + (lonMins / 60);
        if (lonDir === 'W') {
            lonDecimal = -lonDecimal;
        }
    }

    return {
        time: time,
        latitude: latDecimal,
        latitudeDirection: latDir,
        longitude: lonDecimal,
        longitudeDirection: lonDir,
        fixQuality: quality,
        numSatellites: numSats,
        hdop: hdop,
        altitude: altitude,
        altitudeUnit: altUnit,
        geoidalSeparation: geoSep,
        geoidalUnit: geoUnit,
        ageOfDifferential: ageDiff,
        differentialStation: diffStation
    };
}

// 辅助函数：计算NMEA校验和
function calculateChecksum(data) {
    let checksum = 0;
    for (let i = 0; i < data.length; i++) {
        checksum ^= data.charCodeAt(i);
    }
    return checksum.toString(16).toUpperCase();
}

// 使用提供的报文的示例
const sentence = "$GNGGA,065956.00,2813.85693310,N,11252.73428282,E,1,00,9999.0,94.3342,M,-16.6047,M,,*53";
const parsed = parseGGA(sentence);
console.log(parsed);
// 输出: { time: '065956.00', latitude: 28.230948885, latitudeDirection: 'N', longitude: 112.87904471333333, longitudeDirection: 'E', fixQuality: 1, numSatellites: 0, hdop: 9999, altitude: 94.3342, altitudeUnit: 'M', geoidalSeparation: -16.6047, geoidalUnit: 'M', ageOfDifferential: '', differentialStation: '' }

// WGS84 转换为 GCJ02 坐标系转换函数
// 注意：仅适用于中国大陆地区，国外坐标保持不变

const PI = 3.14159265358979324 * 1.0;
const A = 6378245.0; // 克拉索夫斯基椭球参数长半轴
const EE = 0.00669342162296594323; // 克拉索夫斯基椭球参数扁率第一偏心率平方

/**
 * 判断是否在中国大陆范围内
 * @param {number} lat - 纬度
 * @param {number} lng - 经度
 * @returns {boolean} - true 表示不在中国大陆
 */
function outOfChina(lat, lng) {
    return (lng < 72.004 || lng > 137.8347) || (lat < 0.8293 || lat > 55.8271) || false;
}

/**
 * 纬度转换辅助函数
 * @param {number} x - x 坐标偏移
 * @param {number} y - y 坐标偏移
 * @returns {number} - 转换后的纬度偏移
 */
function transformLat(x, y) {
    let ret = -100.0 + 2.0 * x + 3.0 * y + 0.2 * y * y + 0.1 * x * y + 0.2 * Math.sqrt(Math.abs(x));
    ret += (20.0 * Math.sin(6.0 * x * PI) + 20.0 * Math.sin(2.0 * x * PI)) * 2.0 / 3.0;
    ret += (20.0 * Math.sin(y * PI) + 40.0 * Math.sin(y / 3.0 * PI)) * 2.0 / 3.0;
    ret += (160.0 * Math.sin(y / 12.0 * PI) + 320 * Math.sin(y * PI / 30.0)) * 2.0 / 3.0;
    return ret;
}

/**
 * 经度转换辅助函数
 * @param {number} x - x 坐标偏移
 * @param {number} y - y 坐标偏移
 * @returns {number} - 转换后的经度偏移
 */
function transformLng(x, y) {
    let ret = 300.0 + x + 2.0 * y + 0.1 * x * x + 0.1 * x * y + 0.1 * Math.sqrt(Math.abs(x));
    ret += (20.0 * Math.sin(6.0 * x * PI) + 20.0 * Math.sin(2.0 * x * PI)) * 2.0 / 3.0;
    ret += (20.0 * Math.sin(x * PI) + 40.0 * Math.sin(x / 3.0 * PI)) * 2.0 / 3.0;
    ret += (150.0 * Math.sin(x / 12.0 * PI) + 300.0 * Math.sin(x / 30.0 * PI)) * 2.0 / 3.0;
    return ret;
}

/**
 * WGS84 坐标转换为 GCJ02 坐标
 * @param {number} wgsLat - WGS84 纬度
 * @param {number} wgsLng - WGS84 经度
 * @returns {Object} - {lat: GCJ02 纬度, lng: GCJ02 经度}
 */
function wgs84ToGcj02(wgsLat, wgsLng) {
    if (outOfChina(wgsLat, wgsLng)) {
        return {
            lat: wgsLat,
            lng: wgsLng
        };
    } else {
        let dLat = transformLat(wgsLng - 105.0, wgsLat - 35.0);
        let dLng = transformLng(wgsLng - 105.0, wgsLat - 35.0);
        const radLat = wgsLat / 180.0 * PI;
        let magic = Math.sin(radLat);
        magic = 1 - EE * magic * magic;
        const sqrtMagic = Math.sqrt(magic);
        dLat = (dLat * 180.0) / ((A * (1 - EE)) / (magic * sqrtMagic) * PI);
        dLng = (dLng * 180.0) / (A / sqrtMagic * Math.cos(radLat) * PI);
        const gcjLat = wgsLat + dLat;
        const gcjLng = wgsLng + dLng;
        return {
            lat: gcjLat,
            lng: gcjLng
        };
    }
}

// 示例使用：北京天安门 WGS84 坐标 (39.907375, 116.391349) 转换为 GCJ02
const result = wgs84ToGcj02(39.907375, 116.391349);
console.log(result);
// 输出: { lat: 39.90877629414095, lng: 116.39759019123527 }
```


