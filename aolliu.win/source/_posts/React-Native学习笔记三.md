---
title: React Native学习笔记三
date: 2016-08-05 20:27:52
tags:
    - 跨平台开发
---


## ScrollView

- ScrollView是一个通用的可滚动的容器，你可以在其中放入多个组件和视图，而且这些组件并不需要是同类型的。ScrollView不仅可以垂直滚动，还能水平滚动（通过horizontal属性来设置）。
- ScrollView适合用来显示数量不多的滚动元素。放置在ScollView中的所有组件都会被渲染，哪怕有些组件因为内容太长被挤出了屏幕外。如果你需要显示较长的滚动列表，那么应该使用功能差不多但性能更好的ListView组件。

<!--more-->

## ListView
- ListView组件用于显示一个垂直的滚动列表，其中的元素之间结构近似而仅数据不同。
- ListView更适于长列表数据，且元素个数可以增删。和ScrollView不同的是，ListView并不立即渲染所有元素，而是优先渲染屏幕上可见的元素。
- ListView组件必须的两个属性是dataSource和renderRow。dataSource是列表的数据源，而renderRow则逐个解析数据源中的数据，然后返回一个设定好格式的组件来渲染。
- 下面的例子创建了一个简单的ListView，并预设了一些模拟数据。首先是初始化ListView所需的dataSource，其中的每一项（行）数据之后都在renderRow中被渲染成了Text组件，最后构成整个ListView。
- rowHasChanged函数也是ListView的必需属性。这里我们只是简单的比较两行数据是否是同一个数据（===符号只比较基本类型数据的值，和引用类型的地址）来判断某行数据是否变化了。

```
class QRCodePay extends Component{
  // 初始化模拟数据
  constructor(props) {
    super(props);
    const ds = new ListView.DataSource({rowHasChanged: (r1, r2) => r1 !== r2});
    this.state = {
      dataSource: ds.cloneWithRows([
        'John', 'Joel', 'James', 'Jimmy', 'Jackson', 'Jillian', 'Julie', 'Devin'
      ])
    };
  }
  render() {
    return (
        <View style={{flex:1,paddingTop:22}}>
          <ListView dataSource = {this.state.dataSource}
                    renderRow = {(rowData)=> <Text>{rowData}</Text>}
                    style = {{height:40,backgroundColor:'red',marginTop:5}}
          />
        </View>
    );
  }
}
```


# ListView

- 组件ListView组件是React Native核心组件之一,应用十分的广泛,主要是高效的展示列表数据

## 一.基本使用

### 1.首先创建一个ListView.DataSource数据源,然后给它传递一个普通的数据数组
 - 因为数据源在初始化后会改变,所以放到getInitialState方法中,代码如下
 
### 2.使用数据源(dataSource)实例化一个ListView组件,定义一个renderRow回调函数,这个函数会接受数组中的每个数据作为参数,返回一个可渲染的组件(就是listView每一行的item)

```
var LVList = React.createClass({

  // 初始化模拟数据
  getInitialState: function() {
    
    //创建数据源 DataSource是大写  返回新数据的条件是当且仅当两行数据不一样的时候返回新数据
    var ds = new ListView.DataSource({rowHasChanged: (r1, r2) => r1 !== r2});
     this.state = {
            //构造假数据 固定写法,传一个普通的数组
            dataSource: ds.cloneWithRows(['row 1', 'row 2','row 3']),
        };
  },
  render() {
        return (
            <View style={{flex:1}}>
                <ListView
                    dataSource={this.state.dataSource}
                    renderRow={this.renderRow}
                />
            </View>
        );
    },
        //返回每一行的组件
    renderRow: function(rowData: string, sectionID: number, rowID: number) {
    return (
        //添加点击事件
        <TouchableOpacity
            onPress={()=>{alert('点击了第'+sectionID+'组的第'+rowID+'行')}}
        >
            <View style={styles.rowContainer}>
                <View style={styles.row}>
                    <Image style={styles.thumb} source={{uri:Imgs[0]}} />
                    <Text style={{flex:1,fontSize:16,color:'blue',marginLeft:50,}}>
                        {rowData + '测试~'}
                    </Text>
                </View>
            </View>
        </TouchableOpacity>
    );
},
})
```

## 二.常用属性
- 1.ScrolView的相关属性样式都全部继承
- 2.`dataSource`:ListViewDataSource设置ListView的数据源
- 3.`initialListSize`:(number) 设置ListView组件刚刚加载的时候渲染的列表行数,用这个属性确认首页加载的数据数量,而不是花大量的时间渲染大量的数据,提高性能
- 4.`onChangeVisiableRows`: (funtion) (visiableRows,changeRows)=>Void 当可变的行发生变化时回调该方法.
- 5.`onEndReachedThreshold`:(number) 当偏移量达到设置的临界值时调用`ononEndReached`
- 6.`ononEndReached`:(function) 当所有的数据行被渲染之后,并且列表往下进行滚动,一直滚动到距离底部`onEndReachedThreshold`设置的值进行回调该方法,原生的滚动时间进行传递(通过参数的形式)
- 7.`pageSize`:(number) 每一次事件的循环渲染的行数
- 8.`removeClippedSubviews`:(bool) 该属性用于提供大数据列表的滚动性能,该使用的时候需要给每一行(row)的布局挺添加over:`hidden`样式,该属性默认是开启状态.
- 9.`renderFooter`:(function)方法 ()=>renderable 在每次渲染过程中头和尾会重新进行渲染,如果发现该重新绘制的性能开销比较大的时候可以使用StaticContainer容器或者其他合适的组件.
- 10,`renderHeader`:(function)方法 (rowData,sectionID,rowID,highlightRow)=>renderable 该方法有四个参数,其中分别为数据源中的一条数据,分组id,行id,以及标记是否是高亮选中状态信息
- 11.`renderScrollComponent`:(function)方法 (props)=>renderable 该方法可以返回一个可以滚动的组件,默认返回一个ScrollView
- 12.`renderSectionHeader`(function)方法 (sectionData,sectionID)=>renderable 如果设置了该方法,这样会为每一个section渲染一个粘性的header试图,该试图粘性的效果是当刚刚被渲染开始的时候,该试图会处于对应的内容的顶部,然后开始滑动的时候,会跑到屏幕的顶部,知道滑动到下一个section的header,知道被替换
- 13.`renderSeparator` (function)方法 (sectionID,rowID,adjacentRowHighlighted)=>renderable 如果设置了该方法,会在被每一行的下面渲染一个组件作为分割,除了每一个section分组的头部试图前面的最后一行
- 14,`scrollRenderAheadDistance`(number) 进行设置当改行进入屏幕多少像素之后就开始渲染

## 三.高级特性
 - listView有一些高级特性,包括设置每一组的头部,支持设置列表的header以及footer试图,当数据列表滑动到最底部的时候支持`onEndReached`方法回调,设备屏幕列表可见的试图数据发生变化的时候回调`onChangeVisiableRows`以及一些性能方面的优化特性
 - 当需要动态加载非常多的数据的时候,可以使用下面的一些性能优化的方法,让滚动更加平滑:
   - 1).只更新渲染数据变化的那一行,`rowHasChanged`方法会告诉ListView组件是否需要重新渲染当前那一行
   - 2).选择渲染的频率.默认情况下每一个event-loop(事件循环)只会渲染一行(可以同pageSize自定义属性设置),这样可以把大的工作量分割,提高整体的渲染性能.
   

## 四.注意事项
 - 在React Native中ScrollView组件可以使用`stickyHeaderIndices`轻松实现sticky效果,而是用ListView时,使用不生效.

#### 如何实现滚动时每个section header会吸顶呢
 - 在ListView中要实现sticky,需要使用`cloneWithRowsAndSections`方法,将`dataBlob`(object),`sectionIDs`(array),rowIDs(array)三者传进去
   - 1>dataBlob 它包含ListView所需的所有的数据(section header 和rows),在listView渲染数据的时候,使用`getSectionData`和`getRowData`来渲染每一行的数据,dataBlob的key值包含 sectionID + rowID
   - 2>sectionIDs:用于标识每组的seciton
   - 3>rowIDs:用于描述每个section里面的每行数据的位置及是否需要渲染,在ListView渲染时,会先遍历rowIDs 获取到对应的dataBlob数据.
  






















