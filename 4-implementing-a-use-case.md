# Implementing a use case

With the project structure we created, we have a lot of options for how to structure our code, as the application, domain and persistence layers are loosely coupled.

This chapter will give one option, and show you how this looks like in actual code.

## Implementing the domain model

We want to implement the use case of sending money from one account to another.

Let's first create an Account entity in the domain.

### buckpal/domain/Account.java

```java
package buckpal.domain;
public class Account {
  private AccountId id;
  private Money baselineBalance;   // amount of money before activity window
  private ActivityWindow activityWindow;   // all transactions since last save of baseline balance
  
  // constructors and getters omitted
  
  // sums the activities and the baseline
  public Money calculateBalance() { 
    return Money.add(
      this.baselineBalance, 
      this.activityWindow.calculateBalance(this.id)
    );
  }

  // if enough money, adds a withdraw activity of the given amount to activities
  public boolean withdraw(Money money, AccountId targetAccountId) {
    if (!mayWithdraw(money)) {
      return false; 
    }
    Activity withdrawal = new Activity( this.id,
    this.id, targetAccountId, LocalDateTime.now(), money);
    this.activityWindow.addActivity(withdrawal);

    return true; 
  }

  // checks if user has enough money to withdraw the amount
  private boolean mayWithdraw(Money money) { 
    return Money.add(
    this.calculateBalance(), money.negate()).isPositive();
  }

  // adds a deposit activity of the given amount to activities
  public boolean deposit(Money money, AccountId sourceAccountId) {
    Activity deposit = new Activity(
      this.id, 
      sourceAccountId, 
      this.id, 
      LocalDateTime.now(), 
      money
    );

    this.activityWindow.addActivity(deposit);
    return true; 
  }
}
```

Now that we have the entity that respects the business rules, we can go ahead and build the use case.

## A use case in a nutshell

A use case does 4 things:
- take input (input validation isn't the responsibility of the use case)
- validate business rules (check that the action is possible)
- manipulate model state (modify the domain entities, and use output ports most often to persist these changes)
- return output (send infos back to the input adapter that called it)

To create the "send money" use case, we will create its own service class, as each use case must have its own class.

The structure of this class would look like this:

```java
package buckpal.application.service;
@RequiredArgsConstructor
@Transactional
public class SendMoneyService implements SendMoneyUseCase { // implements the input port interface to be called by the input adapters
    // contains the output ports, so that in can get injected with the output adapters
    private final LoadAccountPort loadAccountPort;
    private final UpdateAccountStatePort updateAccountStatePort;

    private final AccountLock accountLock;

    @Override
    public boolean sendMoney(SendMoneyCommand command) {  // This input command will be responsible for the input validation
        // TODO: validate business rules
        // TODO: manipulate model state
        // TODO: return output
    }
}
```

![use case graph](./assets/6.png)

## Validating input

As we said, the use case does not have to validate the input.

But we won't validate the input in each input adapter either because they might make errors, and it will result in duplicated code.

We will therefore validate the input using the input class, in our example `SendMoneyCommand`.

This validation will happen in the constructor, like so:

```java
package buckpal.application.port.in; // validation is part of the input port (inside the hexagon)

@Getter
public class SendMoneyCommand {
  // input model, final (immutable) so no possible changes after validation
  private final AccountId sourceAccountId;
  private final AccountId targetAccountId;
  private final Money money;

  public SendMoneyCommand(AccountId sourceAccountId, AccountId targetAccountId, Money money) {
    this.sourceAccountId = sourceAccountId;
    this.targetAccountId = targetAccountId;
    this.money = money;

    // input validation (will throw if any condition is not respected)
    requireNonNull(sourceAccountId);
    requireNonNull(targetAccountId);
    requireNonNull(money);
    requireGreaterThan(money, 0);
  }
}
```

In Java, we can even make this more concise:

```java
package buckpal.application.port.in;

@Getter
public class SendMoneyCommand extends SelfValidating<SendMoneyCommand> {
    @NotNull
    private final AccountId sourceAccountId;
    @NotNull
    private final AccountId targetAccountId;
    @NotNull
    private final Money money;
    public SendMoneyCommand(AccountId sourceAccountId, AccountId targetAccountId, Money money) {
        this.sourceAccountId = sourceAccountId;
        this.targetAccountId = targetAccountId;
        this.money = money;
        requireGreaterThan(money, 0);
        this.validateSelf();
    }
}
```

For some complex validation, this SelfValidating class might not be sufficient.

We will then have to implement it ourselves, doing something that looks like this:

```java
package shared;
public abstract class SelfValidating < T > {
    private Validator validator;
    public SelfValidating() {
        ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
        validator = factory.getValidator();
    }
    protected void validateSelf() {
        Set < ConstraintViolation < T >> violations = validator.validate((T) this);
        if (!violations.isEmpty()) {
            throw new ConstraintViolationException(violations);
        }
    }
}
```

The input class we created will now protect our use case against bad input, without polluting the use case code with validation code.
