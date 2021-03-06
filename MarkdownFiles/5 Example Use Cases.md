## Example Use Cases 用例案例

This section will demonstrate some example use cases and discuss alternative scenarios.

本节将展示一些用例案例，并讨论替代方案。

### Entity Creation 实体创建

Creating an object from an Entity / Aggregate Root class is the first step of the lifecycle of that entity. The *Aggregate / Aggregate Root Rules & Best Practices* section suggests to **create a primary constructor** for the Entity class that guarantees to **create a valid entity**. So, whenever we need to create an instance of that entity, we should always **use that constructor**.

从Entity / Aggregate Root类创建一个对象是该实体生命周期的第一步。Aggregate / Aggregate Root规则与最佳实践部分建议为Entity类创建一个主构造函数，以保证创建一个有效的实体。因此，每当我们需要创建该实体的实例时，我们应该始终使用该构造函数。

See the `Issue` Aggregate Root class below: 聚合根类`Issue`如下所示：

![image-20211211114837828](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20211211114837828.png)

- This class guarantees to create a valid entity by its constructor.
- 该类保证通过其构造函数创建一个有效的实体。
- If you need to change the `Title` later, you need to use the `SetTitle` method which continues to keep `Title` in a valid state.
- 如果你以后需要改变`Title`，你需要使用`SetTitle`方法，以继续保持`Title`的有效状态。
- If you want to assign this issue to a user, you need to use `IssueManager` (it implements some business rules before the assignment - see the *Domain Services* section above to remember).
- 如果你想把这个`Issue`分配给一个用户，你需要使用`IssueManager`（它在分配前实现了一些业务规则--请参见上面的领域服务部分）。
- The `Text` property has a public setter, because it also accepts null values and does not have any validation rules for this example. It is also optional in the constructor.
- `Text`属性有一个公共`setter`，因为它既可以接受`null`值，而且这个例子中还在没有任何验证规则。它在构造函数中也是可选的参数。

Let's see an Application Service method that is used to create an issue:

让我们看看用于创建一个Issue的应用服务方法：

![image-20211211115016273](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20211211115016273.png)

CreateAsync method; CreateAsync方法：

- Uses the `Issue` **constructor** to create a valid issue. It passes the `Id` using the [`IGuidGenerator`](https://docs.abp.io/en/abp/latest/Guid-Generation) service. It doesn't use auto object mapping here.
- 使用 `Issue`的构造函数来创建一个有效的 `Issue`。它使用 `IGuidGenerator` 服务传递 `Id`。它在这里不使用对象自动映射。
- If the client wants to **assign this issue to a user** on object creation, it uses the `IssueManager` to do it by allowing the `IssueManager` to perform the necessary checks before this assignment.
- 如果客户想要在对象创建时将这个`Issue`分配给一个用户，它就会使用`IssueManager`来做，允许`IssueManager`在分配之前进行必要的检查。
- **Saves** the entity to the database.
- 将实体保存到数据库。
- Finally uses the `IObjectMapper` to return an `IssueDto` that is automatically created by **mapping** from the new `Issue` entity.
- 最后使用`IObjectMapper`返回一个`IssueDto`，这个`IssueDto`是由新`Issue`实体的映射自动创建的。

#### Applying Domain Rules on Entity Creation 在创建实体时应用领域规则

The example `Issue` entity has no business rule on entity creation, except some formal validations in the constructor. However, there maybe scenarios where entity creation should check some extra business rules.

除了构造函数中的一些形式验证外，`Issue`实体的例子在实体创建时没有业务规则验证。然而，也许在某些情况下，实体的创建应该检查一些额外的业务规则。

For example, assume that you **don't want** to allow to create an issue if there is already an issue with **exactly the same `Title`**. Where to implement this rule? It is **not proper** to implement this rule in the **Application Service**, because it is a **core business (domain) rule** that should always be checked.

例如，假设你不希望创建一个已经有一样Title的问题的问题。在哪里实现这个规则？在应用服务中实现这个规则是不合适的，因为它是一个核心业务（领域）规则，应该总是被检查。

This rule should be implemented in a **Domain Service**, `IssueManager` in this case. So, we need to force the Application Layer always to use the `IssueManager` to create a new `Issue`.

这条规则应该在一个领域服务中实现，在本例中就是`IssueManager`。因此，我们需要强制应用层始终使用`IssueManager`来创建一个新的`Issue`。

First, we can make the `Issue` constructor `internal`, instead of `public`:

首先，我们可以把`Issue`的构造函数变成`internal`，而不是 `public`。

![image-20211211115237571](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20211211115237571.png)

This prevents Application Services to directly use the constructor, so they will use the `IssueManager`. Then we can add a `CreateAsync` method to the `IssueManager`:

这使得应用服务无法直接使用构造函数，因此它们会使用`IssueManager`。然后我们可以给`IssueManager`添加一个`CreateAsync`方法，如下所示：

![image-20211211115305136](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20211211115305136.png)

- `CreateAsync` method checks if there is already an issue with the same title and throws a business exception in this case.
- `CreateAsync`方法检查如果已经有一个相同`Title`的`Issue`则在这种情况下抛出一个业务异常。
- If there is no duplication, it creates and returns a new `Issue`.
- 如果没有重复，它会创建并返回一个新的`Issue`。

The `IssueAppService` is changed as shown below in order to use the `IssueManager`'s `CreateAsync` method:

为了使用`IssueManager`的`CreateAsync`方法，`IssueAppService`被修改如下：

![image-20211211115425137](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20211211115425137.png)

##### Discussion: Why is the `Issue` not saved to the database in `IssueManager`? 讨论：为什么在`IssueManager`中没有将`Issue`保存到数据库中？

You may ask "Why didn't IssueManager save the Issue to the database?". We think it is the responsibility of the Application Service.

你可能会问 "为什么`IssueManager`没有把`Issue`保存到数据库中？"。我们认为这是应用服务的职责。

Because, the Application Service may require additional changes/operations on the `Issue` object before saving it. If Domain Service saves it, then the Save operation is duplicated;

因为，应用服务可能在保存`Issue`对象之前需要对其进行额外的更改/操作。如果领域服务保存它，那么保存操作是重复的。

- It causes performance lost because of double database round trip.
- 两次的数据库往返导致性能损失。
- It requires explicit database transaction that covers both operations.
- 它需要明确涵盖这两种操作的数据库事务。
- If additional actions cancel the entity creation because of a business rule, the transaction should be rolled back in the database.
- 如果额外的操作因为业务规则而取消了实体的创建，该事务应该在数据库中被回滚。

When you check the `IssueAppService`, you will see the advantage of **not saving** `Issue` to the database in the `IssueManager.CreateAsync`. Otherwise, we would need to perform one *Insert* (in the `IssueManager`) and one *Update* (after the Assignment).

当你查看`IssueAppService`时，你会发现在IssueManager.CreateAsync中不把`Issue`保存到数据库的好处。否则，我们将需要执行一次插入（在`IssueManager`中）和一次更新（在赋值之后）。

##### Discussion: Why is the duplicate `Title` check not implemented in the Application Service? 讨论：为什么不在应用服务中实现重复`Title`的检查？

We could simply say "Because it is a `core domain logic` and should be implemented in the Domain Layer". However, it brings a new question "**How did you decide** that it is a core domain logic, but not an application logic?" (we will discuss the difference later with more details).

简单来说说 "因为它是核心领域逻辑，应该在领域层实现"。然而，这又带来了一个新的问题："你是如何决定它是一个核心领域逻辑，而不是一个应用逻辑的？" (我们将在后面详细讨论其中的区别）。

For this example, a simple question can help us to make the decision: "If we have another way (use case) of creating an issue, should we still apply the same rule? Is that rule should always be implemented". You may think "Why do we have a second way of creating an issue?". However, in real life, you have;

对于这个例子，一个简单的提问可以帮助我们做出决定。"如果我们有另一种创建`Issue`的方式（用例），我们是否还应该应用同样的规则？那条规则是否应该总是被执行"。你可能会想 "为什么我们会有第二种创建`Issue`的方式？"。然而，在现实生活中，确实存在。

- **End users** of the application may create issues in your application's standard UI.
- 应用程序的终端用户可能会在你的应用程序的标准用户界面中创建`Issue`。
- You may have a second **back office** application that is used by your own employees and you may want to provide a way of creating issues (probably with different authorization rules in this case).
- 你可能有由你自己的员工使用的第二个后台应用程序，你可能想提供一种创建`Issue`的方式（在这种情况下可能有不同的授权规则）。
- You may have an HTTP API that is open to **3rd-party clients** and they create issues.
- 你可能有一个对第三方客户端开放的HTTP API，他们会创建`Issue`。
- You may have a **background worker** service that do something and creates issues if it detects some problems. In this way, it will create an issue without any user interaction (and probably without any standard authorization check).
- 你可能有一个处理某些事情的后台处理服务，如果它检测到一些问题时就会创建`Issue`。通过这种方式，它将在没有任何用户交互的情况下创建一个`Issue`（可能也没有任何标准的授权检查）。
- You may have a button on the UI that **converts** something (for example, a discussion) to an issue.
- 你可能在用户界面上有一个按钮，将某些东西（例如讨论）转换为一个`Issue`。

We can give more examples. All of these are should be implemented by **different Application Service methods** (see the Multiple Application Layers section below), but they **always** follow the rule: Title of the new issue can not be same of any existing issue! That's why this logic is a **core domain logic**, should be located in the Domain Layer and **should not be duplicated** in all these application service methods.

我们可以举出更多的例子。所有这些都应该由不同的应用服务方法来实现（见下面的多应用层部分），但它们总是遵循这个规则：新`Issue`的`Title`不能与任何现有`Issue`的相同！这就是为什么这个逻辑是一个核心的领域逻辑，应该位于领域层，不应该在所有这些应用服务方法中重复出现。