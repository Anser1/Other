## 11. Softmax与Sigmoid
-- 
Softmax：更像是在解决唯一性的问题。例：从一堆概率中选出最大的来做为最终的预测。
<img width="605" height="272" alt="image" src="https://github.com/user-attachments/assets/b0dff41b-f6b3-4715-845d-c71dded0e43c" />

其标签中存在竞争关系，即单标签多分类；
Softmax：类别之间强制竞争
- “在 {猫, 狗, 鸟} 里，你到底是哪一个？”
适合：
- 单标签多分类（只能选一个）

Sigmoid：是在解决是与否的问题。例：对于多个物体判断每个物体是否属于该类别。
<img width="566" height="224" alt="image" src="https://github.com/user-attachments/assets/fd2024c4-d9ca-4561-b955-1022f8c3abe1" />

各标签之间互不影响，即多标签多分类；
Sigmoid：类别之间不竞争
- “这是不是猫？”
- “这是不是狗？”
- 两个问题互不影响
适合：
- 多标签任务（一张图既有猫又有狗）
- 二分类（正/负）

