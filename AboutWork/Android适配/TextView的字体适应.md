TextView展示区域有限而文字无限长的需求
TextView提供的AutoSize功能
具体用法
1. 设置TextView固定宽高
```
 android:layout_width="35dp"
 android:layout_height="14dp"
```
2. 三个属性自适应
```
app:autoSizeMaxTextSize="12dp"
app:autoSizeMinTextSize="4dp"
app:autoSizeTextType="uniform"
```
注意，以上单位是dp