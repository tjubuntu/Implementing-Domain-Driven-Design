## Domain Logic & Application Logic 领域逻辑&应用逻辑

As mentioned before, Business Logic in the Domain Driven Design is split into two parts (layers): Domain Logic and Application Logic:

如前所述，领域驱动设计中的业务逻辑被分成两部分（层）：领域逻辑和应用逻辑：

![image-20211214114143012](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20211214114143012.png)

Domain Logic consists of the Core Domain Rules of the system while Application Logic implements application specific Use Cases.

领域逻辑由系统的核心领域规则组成，而应用逻辑则实现具体的应用案例。

While the definition is clear, the implementation may not be easy. You may be undecided which code should stand in the Application Layer, which code should be in the Domain Layer. This section tries to explain the differences.

虽然定义很清楚，但实施起来可能并不容易。你可能无法决定哪些代码应该放在应用层，哪些代码应该在领域层。本章节试图解释这些区别。