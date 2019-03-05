工具类：
```Java
import java.util.ArrayList;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

public class AssmTreeUtil {
	public static List<TreeNode> assmTree(List<TreeNode> singleTreeNodes) {
		if(singleTreeNodes == null || singleTreeNodes.isEmpty()) {
			return null;
		}

		List<TreeNode> nodeTrees = new ArrayList<TreeNode>();
		Map<String,TreeNode> treeMap = new LinkedHashMap<String,TreeNode>();
		for(TreeNode node : singleTreeNodes){
			TreeNode treeNode = new TreeNode();
			treeNode.setNodeId(node.getNodeId());
			treeNode.setNodeName(node.getNodeName());
			treeNode.setPid(node.getPid());
			treeMap.put(node.getNodeId(), treeNode);
		}
		for(String nodeId : treeMap.keySet()){
			TreeNode treeNode = treeMap.get(nodeId);
			String pid = treeNode.getPid();
			if(pid==null || pid.length()==0 || !treeMap.containsKey(pid)){
				treeNode.setPid("");
				nodeTrees.add(treeNode);
			}else{
				TreeNode parentTreeNode = treeMap.get(pid);
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

bean类：
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
