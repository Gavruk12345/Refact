# Refact

Цей проект створено за допомогою Unity. Нижче наведено інструкції з встановлення, запуску та використання проекту, а також секції з прикладами коду до і після змін.

## Вимоги

Перед початком переконайтеся, що у вас встановлено:

- [Unity Hub](https://unity.com/download)
- Остання версія Unity (перевірте у проекті яку саме версію використовувати)

## Установка

1. Клонування репозиторію:

2. Відкрийте Unity Hub і натисніть кнопку `Add`.

3. Виберіть папку з проектом, який ви тільки що клонували.

4. Відкрийте проект в Unity.

## Запуск

1. В Unity, виберіть сцену, з якої ви хочете почати (зазвичай це `Assets/Scenes/Main.unity`).

2. Натисніть кнопку `Play` у верхній частині редактора Unity.

## Деплоймент

Для побудови проекту:

1. Перейдіть до `File` > `Build Settings`.

2. Виберіть платформу (наприклад, `PC, Mac & Linux Standalone`, `Android`, `iOS`).

3. Натисніть `Build` і виберіть директорію для збереження білду.


---

## Шаблонні секції з прикладами коду

### Код зі "запахом" : Великі методи
Запах: Метод Update() занадто великий і виконує багато завдань.

PlayerController.cs

**До:**
```csharp
protected override void Update()
{
    if (controlEnabled)
    {
        move.x = Input.GetAxis("Horizontal");
        if (jumpState == JumpState.Grounded && Input.GetButtonDown("Jump"))
            jumpState = JumpState.PrepareToJump;
        else if (Input.GetButtonUp("Jump"))
        {
            stopJump = true;
            Schedule<PlayerStopJump>().player = this;
        }
    }
    else
    {
        move.x = 0;
    }
    UpdateJumpState();
    base.Update();
}

```

**Після**
```csharp
protected override void Update()
{
    HandleInput();
    UpdateJumpState();
    base.Update();
}

void HandleInput()
{
    if (controlEnabled)
    {
        move.x = Input.GetAxis("Horizontal");
        if (jumpState == JumpState.Grounded && Input.GetButtonDown("Jump"))
            jumpState = JumpState.PrepareToJump;
        else if (Input.GetButtonUp("Jump"))
        {
            stopJump = true;
            Schedule<PlayerStopJump>().player = this;
        }
    }
    else
    {
        move.x = 0;
    }
}
```
Опис: Розбиття методу Update() на менші методи, такі як HandleInput(), що підвищує зрозумілість та легкість підтримки коду.

### Код зі "запахом" : Використання магічних чисел

Запах: Використання магічних чисел у методі ComputeVelocity() для перевірки значення move.x.

PlayerController.cs

**До:**
```csharp
protected override void ComputeVelocity()
{
    if (jump && IsGrounded)
    {
        velocity.y = jumpTakeOffSpeed * model.jumpModifier;
        jump = false;
    }
    else if (stopJump)
    {
        stopJump = false;
        if (velocity.y > 0)
        {
            velocity.y = velocity.y * model.jumpDeceleration;
        }
    }

    if (move.x > 0.01f)
        spriteRenderer.flipX = false;
    else if (move.x < -0.01f)
        spriteRenderer.flipX = true;

    animator.SetBool("grounded", IsGrounded);
    animator.SetFloat("velocityX", Mathf.Abs(velocity.x) / maxSpeed);

    targetVelocity = move * maxSpeed;
}


```

**Після**
```csharp
private const float MovementThreshold = 0.01f;

protected override void ComputeVelocity()
{
    if (jump && IsGrounded)
    {
        velocity.y = jumpTakeOffSpeed * model.jumpModifier;
        jump = false;
    }
    else if (stopJump)
    {
        stopJump = false;
        if (velocity.y > 0)
        {
            velocity.y = velocity.y * model.jumpDeceleration;
        }
    }

    if (move.x > MovementThreshold)
        spriteRenderer.flipX = false;
    else if (move.x < -MovementThreshold)
        spriteRenderer.flipX = true;

    animator.SetBool("grounded", IsGrounded);
    animator.SetFloat("velocityX", Mathf.Abs(velocity.x) / maxSpeed);

    targetVelocity = move * maxSpeed;
}

```
Опис: Винесення магічних чисел у константи підвищує зрозумілість та дозволяє легше змінювати значення при потребі.

### Код зі "запахом" : Великі методи

Метод FixedUpdate занадто великий і містить кілька логічних блоків, які можна виділити в окремі методи для підвищення читабельності та повторного використання коду.



**До:**
```csharp

protected virtual void FixedUpdate()
{
    if (velocity.y < 0)
        velocity += gravityModifier * Physics2D.gravity * Time.deltaTime;
    else
        velocity += Physics2D.gravity * Time.deltaTime;

    velocity.x = targetVelocity.x;

    IsGrounded = false;

    var deltaPosition = velocity * Time.deltaTime;

    var moveAlongGround = new Vector2(groundNormal.y, -groundNormal.x);

    var move = moveAlongGround * deltaPosition.x;

    PerformMovement(move, false);

    move = Vector2.up * deltaPosition.y;

    PerformMovement(move, true);
}

```

**Після**
```csharp
protected virtual void FixedUpdate()
{
    ApplyGravity();
    AdjustVelocity();
    IsGrounded = false;

    var deltaPosition = velocity * Time.deltaTime;
    MoveAlongGround(deltaPosition);
    MoveVertically(deltaPosition);
}

private void ApplyGravity()
{
    if (velocity.y < 0)
        velocity += gravityModifier * Physics2D.gravity * Time.deltaTime;
    else
        velocity += Physics2D.gravity * Time.deltaTime;
}

private void AdjustVelocity()
{
    velocity.x = targetVelocity.x;
}

private void MoveAlongGround(Vector2 deltaPosition)
{
    var moveAlongGround = new Vector2(groundNormal.y, -groundNormal.x);
    var move = moveAlongGround * deltaPosition.x;
    PerformMovement(move, false);
}

private void MoveVertically(Vector2 deltaPosition)
{
    var move = Vector2.up * deltaPosition.y;
    PerformMovement(move, true);
}

```

### Код зі "запахом" : 

**До:**
```csharp


```

**Після**
```csharp

```



