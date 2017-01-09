# vue-datetime
使用vue编写的时间组件，小巧实用，支持vue1.0，vue2.0  
```
v1.0 功能:  
1.支持同时展开多个日期选择框  
2.支持单击选中和取消，可配置单选和多选  
3.支持双击启动连续选择，支持正向连续，逆向连续和跳跃不可选日期  
4.支持在日期选择框内直接切换月份  
5.支持初始化不可点击日期(剩余的可选择)  
6.支持初始化已选择日期(已选择日期高亮)  
7.支持初始化可选择日期(剩余的不可选择)  
8.同时初始化不可点击和可点击日期，将以可点击日期为准  
```

```
v1.1:  
1.修复已知bug  
```

```
v1.2:  
1.重构代码，使代码更优雅和简洁  
2.性能优化
3.完善功能，使多时间框和原地切换时间共存
4.支持不传入options 和datelist的使，默认显示当前日期并且可原地切换日期
```

```
v1.3:  
1.增加年份和月份的快速选择
```

用法：`<edit-time :datelist='datelist' :options='options'></edit-time>`

    datelist = [
        {
            year: undefined, // 日期初始年, 默认当前年
            month: undefined, // 日期初始月, 默认当前月
            multiSelect: true, // 日期是否可多选
            switch: false // 当前日期框是否支持切换年月份
        }
    ]
    options = {
    disable: [], // 不可选日期，格式: '2016-01-01'
    // enable: [], // 可选日期，格式: '2016-01-01'，enable和disable只能有一个，如果都有默认用enable
    selected: [], // 已选择的day，格式: '2016-01-01'
    callback: undefined // 点击日期回调函数, callback(selectDateList)
    }

图示:
[github图示有问题，请点击这查看](http://blog.csdn.net/xingyeyongheng/article/details/54289241)  
![图示](http://github-image.oss-cn-hangzhou.aliyuncs.com/tmpdir--17_1_9_21_55_32.gif)
