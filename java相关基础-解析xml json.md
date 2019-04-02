# XML的解析方式分为四种：1、DOM解析；2、SAX解析；3、JDOM解析；4、DOM4J解析
## Dom4j解析xml文件-递归遍历所有节点和属性
```java
<?xml version="1.0" encoding="UTF-8"?>
<root>
	<user editor="chenleixing" date="2015-02-15">
		<name>张三</name>
		<year>24</year>
		<sex>男</sex>
	</user>
	<user editor="zhangxiaochao" date="2015-02-15">
		<name>李四</name>
		<year>24</year>
		<sex>女</sex>
	</user>
</root>
```
创建File,获取
```java
        /**
	 * 获取文件的document对象，然后获取对应的根节点
	 * @author chenleixing
	 */
	 
	@Test
	public void testGetRoot() throws Exception{
		SAXReader sax=new SAXReader();//创建一个SAXReader对象
		File xmlFile=new File("d:\\test.xml");//根据指定的路径创建file对象
		Document document=sax.read(xmlFile);//获取document对象,如果文档无节点，则会抛出Exception提前结束
		Element root=document.getRootElement();//获取根节点
		this.getNodes(root);//从根节点开始遍历所有节点
 
	}
```
	
从指定节点开始，递归遍历所有节点和属性
```java
        /**
	 * 从指定节点开始,递归遍历所有子节点
	 * @author chenleixing
	 */
	 
	public void getNodes(Element node){
		System.out.println("--------------------");
		//当前节点的名称、文本内容和属性
		System.out.println("当前节点名称："+node.getName());//当前节点名称
		System.out.println("当前节点的内容："+node.getTextTrim());//当前节点名称
		List<Attribute> listAttr=node.attributes();//当前节点的所有属性的list
		for(Attribute attr:listAttr){//遍历当前节点的所有属性
			String name=attr.getName();//属性名称
			String value=attr.getValue();//属性的值
			System.out.println("属性名称："+name+"属性值："+value);
		}
		
		//递归遍历当前节点所有的子节点
		List<Element> listElement=node.elements();//所有一级子节点的list
		for(Element e:listElement){//遍历所有一级子节点
			this.getNodes(e);//递归
		}
	}
```
## SAX
全称Simple API for XML，是一种以事件驱动的XMl API
> SAX的优点：
   解析速度快
   占用内存少

> SAX的缺点：
无法知道当前解析标签（节点）的上层标签，及其嵌套结构，仅仅知道当前解析的标签的名字和属性，要知道其他信息需要程序猿自己编码
只能读取XML，无法修改XML
无法随机访问某个标签（节点）
> SAX解析适用场合 
对于CPU资源宝贵的设备，如Android等移动设备
对于只需从xml读取信息而无需修改xml

## Android推荐的Pull解析方式.
PULL解析器的运行方式和SAX类似，都是基于事件的模式
```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        TextView tv_text = (TextView) findViewById(R.id.tv_text);
        try {
            //得到XML解析器
            XmlPullParser pullParser = Xml.newPullParser();
            InputStream is = getAssets().open("activity_main.xml");
            pullParser.setInput(is, "utf-8");
            //得到事件类型
            int eventType = pullParser.getEventType();
            //文档的末尾
            //遍历内部的内容
            StringBuilder stringBuilder = new StringBuilder();
            while (eventType != XmlPullParser.END_DOCUMENT) {
                String name = pullParser.getName();
                if (!TextUtils.isEmpty(name))
                    if (eventType == XmlPullParser.START_TAG) {
                        String attributeValue = pullParser.getAttributeValue(null, "id");
                        attributeValue = attributeValue.substring(attributeValue.indexOf("/") + 1, attributeValue.length());

                        stringBuilder.append("name====");
                        stringBuilder.append(name);
                        stringBuilder.append("\t\tid====");
                        stringBuilder.append(attributeValue);
                        stringBuilder.append("\n\n");
                    }
                eventType = pullParser.next();//读取下一个标签
            }
            tv_text.setText(stringBuilder.toString());

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```
