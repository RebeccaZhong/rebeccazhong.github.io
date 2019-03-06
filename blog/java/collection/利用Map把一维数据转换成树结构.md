待改进：
1. 前面加一段文字，介绍一下这篇博客是干什么的
2. 代码要有注释
3. 简要说明一下你的实现逻辑 有必要画图

工具类：
```Java
import java.util.ArrayList;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

public class AssmTreeUtil {
	/**
	* 组装树结构数据方法
	*/
	public static List<TreeNode> assmTree(List<TreeNode> singleTreeNodes) {
		// 判断排序数据是否为空
		if(singleTreeNodes == null || singleTreeNodes.isEmpty()) {
			return null;
		}
		// 用有序Map把传参组装起来
		Map<String,TreeNode> nodeId2treeNodes = new LinkedHashMap<String,TreeNode>();
		for(TreeNode node : singleTreeNodes){
			TreeNode treeNode = new TreeNode();
			treeNode.setNodeId(node.getNodeId());
			treeNode.setNodeName(node.getNodeName());
			treeNode.setPid(node.getPid());
			nodeId2treeNodes.put(node.getNodeId(), treeNode);
		}
		// 用来保存组装好的数据, 作为返回值
		List<TreeNode> nodeTrees = new ArrayList<TreeNode>();
		// 遍历组装好的有序Map
		for(String nodeId : nodeId2treeNodes.keySet()){
			TreeNode treeNode = nodeId2treeNodes.get(nodeId);
			String pid = treeNode.getPid();
			// 如果父节点为空或 没有以此父节点???
			if(pid==null || pid.length()==0 || !nodeId2treeNodes.containsKey(pid)){
				treeNode.setPid("");
				nodeTrees.add(treeNode);
			}else{
				TreeNode parentTreeNode = nodeId2treeNodes.get(pid);
				if(parentTreeNode.getChildren()==null){
					parentTreeNode.setChildren(new ArrayList<TreeNode>());
				}
				parentTreeNode.getChildren().add(treeNode);
			}
		}
		return nodeTrees;
	}
}
```

TreeNode bean类：
```java
import java.util.List;

public class TreeNode {
	/**
	 * 父节点ID
	 */
	private String pid;
	/**
	 * 节点ID
	 */
	private String nodeId;
	/**
	 * 节点名称
	 */
	private String nodeName;
	/**
	 * 子节点
	 */
	private List<TreeNode> children;
	public String getPid() {
		return pid;
	}
	public void setPid(String pid) {
		this.pid = pid;
	}
	public String getNodeId() {
		return nodeId;
	}
	public void setNodeId(String nodeId) {
		this.nodeId = nodeId;
	}
	public String getNodeName() {
		return nodeName;
	}
	public void setNodeName(String nodeName) {
		this.nodeName = nodeName;
	}
	public List<TreeNode> getChildren() {
		return children;
	}
	public void setChildren(List<TreeNode> children) {
		this.children = children;
	}

}
```

**小结：**
实现相关功能的，也有其它例子，但都是用多个ArrayList的，代码比较长。利用LinkedHashMap能更好的实现。
