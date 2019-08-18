#### 深度优先遍历
```
// 栈实现
let tree = {
    content: '1',
    child: [
        { 
            content: '1-1',
            child: [{ content: '1-1-1' }, { content: '1-1-2' }]
        },
        { content: '1-2' }
    ]
}
function deepVisited(root) {
    let stack = [],    // 存储遍历过程的节点
        nodeList = []; // 遍历的结果
    stack.push(root);
    while (stack.length) {
        let node = stack.pop();
        nodeList.push(node);
        if (node.child) {
            for (let index=node.child.length-1; index >= 0; index--) {
                stack.push(node.child[index]);
            }
        }
    }
    return nodeList;
}
```
#### 广度优先遍历
```
// 队列
let tree = {
    content: '1',
    child: [
        { 
            content: '1-1',
            child: [{ content: '1-1-1' }, { content: '1-1-2' }]
        },
        { content: '1-2' }
    ]
}
function breadthVisited(root) {
    let nodeList = [],
        queue = [];
    queue.push(root);
    while (queue.length) {
        let node = queue.shift();
        nodeList.push(node);
        if (node.child) {
            for (let index=0; index<node.child.length; index++) {
                queue.push(node.child[index]);
            }
        }
    }
    return nodeList;
}
```
