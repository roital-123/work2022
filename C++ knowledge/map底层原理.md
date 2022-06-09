# map底层原理（set同理）

## 红黑树（RBTree）

- 红黑树性质：
  - 根节点为黑色
  - 叶子节点（无论红黑，**逻辑**上认为它的左、右子节点为空的黑节点NIL（**逻辑叶子**））
  - 红节点，左右孩子节点只能为黑节点
  - 从任意一个节点出发，到**其下的NIL节点**经过的黑节点个数相同

- 添加节点**c** ---- **cur**（默认每次新增的节点为**红色节点**）

  - 每次新增的节点只会在叶子节点（**物理叶子**）上；
  - 初次添加，树为空；当前节点变色
  - 父节点**p** ---- **parent**为黑色：直接添加（没有破坏红黑树性质）
  - p为红色（一定存在祖父节点**g** ---- **grandParent**且为黑，因为树根必须黑）
  - **（1）p**是**g**的左孩子
    - 叔父节点**u** ---- **uncle**存在且为红
      - **g**变红色，**p和u**变黑色；递归向上
    - u不存在或者存在且为黑（不存在在逻辑上就是**黑NIL**）
      - **c**是**p**的左孩子，**g**右旋，**p**变黑色变为当前的根，**g**变红色（结束，不需要递归）
      - **c**是**p**的右孩子，先对**p**左旋，变为上面的情况，做相同的操作
  - **（2）p**是**g**的右孩子 （就是情况1的逆向操作）
    - 叔父节点**u** ---- **uncle**存在且为红
      - 。。。
      - 。。。
    - u不存在或者存在且为黑（不存在在逻辑上就是**黑NIL**）

- 删除节点

  - 删除节点太复杂了 😭😭
  - [删除节点分类讨论](https://blog.csdn.net/qq_40843865/article/details/102498310)
    - 其：组合5&6中 (a)图中后继节点为黑色时，应转为组合2或4

- 实现思路：

  ```c++
  // RBTree.h
  // 只含有插入情况：
  enum COLOR
  {
      RED,
      BLACK
  };
  
  template<class K, class V>
  struct RBTreeNode
  {
      RBTreeNode(const K& _key, const V& _val, COLOR _color = RED)
          : key(_key), value(_val), color(_color), left(nullptr), right(nullptr), parent(nullptr)
      {}
      K key;
      V value;
      COLOR color;
      RBTreeNode<K, V>* left;
      RBTreeNode<K, V>* right;
      RBTreeNode<K, V>* parent;
  };
  
  template<class K, class V>
  class RBTree
  {
  public:
      typedef RBTreeNode<K, V> Node;
      typedef RBTreeNode<K, V>* pNode;
  
      RBTree() : root(nullptr) {}
  
      void Insert(const K& key, const V& val)
      {       
          if (root == nullptr)
          {
              root = new RBTreeNode<K, V>(key, val, BLACK);
              return;
          }
          pNode parent = nullptr;
          pNode cur = root;
          while (cur) {
              if (cur->key > key) parent = cur, cur = cur->left;
              else if (cur->key < key) parent = cur, cur = cur->right;
              else return;	// 插入失败，可以通过返回值修改
          }
  
          cur = new RBTreeNode<K, V>(key, val);
          if (key < parent->key) parent->left = cur;
          else parent->right = cur;
          cur->parent = parent;
  
          while (parent && parent->color == RED)
          {
              pNode grandP = parent->parent;
              if (parent == grandP->left)
              {
                  pNode uncle = grandP->right;
                  if (uncle && uncle->color == RED)
                  {
                      parent->color = BLACK;
                      uncle->color = BLACK;
                      grandP->color = RED;
  
                      cur = grandP;
                      parent = grandP->parent;
                  }
                  else
                  {
                      if (cur == parent->right)
                      {
                          RotateL(parent);
                          swap(cur, parent);
                      }
                      RotateR(grandP);
                      parent->color = BLACK;
                      grandP->color = RED;
                      break;
                  }
              }
              else
              {
                  pNode uncle = grandP->left;
                  if (uncle && uncle->color == RED)
                  {
                      parent->color = BLACK;
                      uncle->color = BLACK;
                      grandP->color = RED;
  
                      cur = grandP;
                      parent = grandP->parent;
                  }
                  else
                  {
                      if (cur == parent->left)
                      {
                          RotateR(parent);
                          swap(cur, parent);
                      }
                      RotateL(grandP);
                      parent->color = BLACK;
                      grandP->color = RED;
                      break;
                  }
              }
          }
          
        	// 这一步很重要
          root->color = BLACK;
      }
  
      pNode getRoot() const { return root; }
  
  private:
      void RotateL(pNode pParent)
      {
          pNode subR = pParent->right;
          pNode subRL = subR->left;
  
          subR->left = pParent;
          pParent->right = subRL;
          if (subRL) subRL->parent = pParent;
  
          pNode pParentP = pParent->parent;
          subR->parent = pParentP;
          pParent->parent = subR;
          if (pParent == root) root = subR;
          if (pParentP)
          {
              if (pParentP->left == pParent) pParentP->left = subR;
              else pParentP->right = subR;
          }
      }
  
      void RotateR(pNode pParent)
      {
          pNode subL = pParent->left;
          pNode subLR = subL->right;
          
          subL->right = pParent;
          pParent->left = subLR;
          if (subLR) subLR->parent = pParent;
  
          pNode pParentP = pParent->parent;
          subL->parent = pParentP;
          pParent->parent = subL;
          if (pParent == root) root = subL;
          if (pParentP)
          {
              if (pParentP->left == pParent) pParentP->left = subL;
              else pParentP->right = subL;
          }
      }
  
      pNode root;
  };
  
  template<class K, class V>
  void inOrder(RBTreeNode<K, V> *node)
  {
      if (node == nullptr) return;
      inOrder(node->left);
      cout << node->value << ' ';
      inOrder(node->right);
  }
  
  template<class K, class V>
  void preOrder(RBTreeNode<K, V> *node)
  {
      if (node == nullptr) return;
      cout << node->value << ' ';
      preOrder(node->left); 
      preOrder(node->right);
  }
  ```
  
  































