一、Dom4j解析xml文件-递归遍历所有节点和属性
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

创建File,获取
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
从指定节点开始，递归遍历所有节点和属性
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
