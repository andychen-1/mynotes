

默认情况下 Qt6 Android 的主题默认指定为 Google Material

``` js
import QtQuick.Controls.Material
// 由于主题属性默认向下传递（继承），建议在顶级组件中添加，
// 可以一次性修改所有容器包含的标准组件，例如 Button, TextField 等
Page {
	// Material.accent 为主题强调色，通常是控件焦点显示时的颜色
	Material.accent: AppTheme.palette.primary
	
	// ...
	BusyIndicator {
		anchors.fill: parent
		anchors.centerIn: parent
		running: control.isLoadingMountpoints
	}
	// ...
}
```