# BFS&DFS

## BFS

```JAVA
// BFS
    public int minDepth(TreeNode root) {
        Queue<TreeNode> q = new LinkedList<TreeNode>();
        if(root == null) return 0;
        if(root.left == null && root.right == null) return 1;

        q.offer(root);
        int res = 1;
        while(!q.isEmpty()){
            int sz = q.size();

            for(int i = 0; i < sz; i++){
                TreeNode t = q.poll();
                if(t.left == null && t.right == null){
                    return res;
                }

                if(t.left != null){
                    q.offer(t.left);
                }
                if(t.right!=null){
                    q.offer(t.right);
                }
            }

            res ++;

        }
        return res;
    }

```

## DFS

```java
  public int minDepth(TreeNode root) {
         if(root == null) return 0;

         if(root.left == null && root.right == null) return 1;

         int res = Integer.MAX_VALUE;
         if(root.left != null ) res = Math.min(minDepth(root.left), res);
         if(root.right != null ) res = Math.min(minDepth(root.right), res);

         return res +1;
     }
```
