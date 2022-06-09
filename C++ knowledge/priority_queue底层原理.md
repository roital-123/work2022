# 堆的底层原理

- 底层容器一般都是vetor，默认是大根堆

  ```c
  namespace pri_que{
    template<typename T>
    struct less{
      bool operator()(const T& left, const T& right)
  		{
  			return left < right;
  		}
    };
    
    template<typename T>
    struct greater{
      bool operator()(const T& left, const T& right)
  		{
  			return left > right;
  		}
    };
    
    template<typename T, typename Container = vector<T>, typename Comp = less<T>>
    class Priority_queue{
    public:
    	void push(const T& x) {
        c.push_back(x);
        AdjustUp(c.size()-1);
      }
      
      void pop(){
        swap(c[0], c[c.size()-1]);
        c.pop_back();
        AdjustDown(0);
      }
      
      T& top() {
        return c[0];
      }
      
      size_t size() {
        return c.size();
      }
      
      bool empty() {
        return c.empty();
      }
      
    private:
      Container c;
      
      void AdjustUp(int cur) {
        Comp cmp;
        while(cur != 0) {
          int pa = (cur-1)/2;
          if(cmp(pa, cur)) {
            swap(c[pa], c[cur]);
            cur = pa;
          }
        }
      }
      
      void AdjustDown(int cur) {
        int child = 2*cur + 1;
  			Comp cmp;
  			while (child < c.size())
  			{
  				if (child+1 < c.size() && cmp(c[child], c[child+1])) {
  					++child;
  				}
  				if (cmp(c[cur], c[child])) {
  					swap(c[child], c[cur]);
  					cur = child;
  					child = 2*cur + 1;
  				}
  				else break;
  			}
      }
    };
  }
  ```
  
  
  
- priority_queue 基本API

  ```c++
  /*
  priority_queue<int, vector<int>, less<int>> bq; // 大根堆
  priority_queue<int, vector<int>, greater<int>> sq; // 小根堆
  
  bq.push();
  bq.top();
  bq.pop();
  */
  ```

  