 Глава 2: Погружение в C++ и Game Framework
Добро пожаловать в реальный Unreal Engine 5. Если вы до сих пор ограничивались Blueprints, пора переходить на C++. Это не просто прихоть – это ключ к производительности, гибкости и возможностям, недоступным в визуальном скриптинге.

Здесь мы разберем:
✅ Как работает игровой фреймворк UE5
✅ Как писать код на C++ под Unreal
✅ Создадим свой первый Actor и Pawn
✅ Разберем, как взаимодействуют компоненты, объекты и классы

📌 Дисклеймер: Если у вас нет опыта в C++, пора подтянуть его. Unreal C++ – это особая экосистема с умными указателями, макросами и объектной системой UE.

🔥 2.1. Архитектура Game Framework в UE5
Перед тем как писать код, давайте разберем основные классы игрового фреймворка.

🔹 GameModeBase – управляет логикой режима игры (например, режим "Королевская битва" или "Шутер 5v5").
🔹 PlayerController – обрабатывает ввод игрока (например, движение камеры).
🔹 Pawn – игровая сущность, которой управляет игрок или AI.
🔹 Character – специальный вид Pawn с готовыми компонентами для передвижения.
🔹 Actor – базовый класс всех объектов на сцене.

💡 Важно: В UE5 есть Garbage Collector (GC), поэтому НЕ используйте new и delete. Работайте через UObjects и TSharedPtr / TWeakObjectPtr.

🔹 2.2. Создание первого C++ Actor
1️⃣ Создаем C++ Класс
Открываем Unreal Editor → Tools → New C++ Class
Выбираем Actor → Назовем его MovingPlatform
Открываем MovingPlatform.h и MovingPlatform.cpp.
2️⃣ Добавляем компонент Static Mesh
📌 MovingPlatform.h

cpp
Копировать
Редактировать
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "MovingPlatform.generated.h"

UCLASS()
class MYPROJECT_API AMovingPlatform : public AActor
{
    GENERATED_BODY()

public:	
    AMovingPlatform();

protected:
    virtual void BeginPlay() override;

public:	
    virtual void Tick(float DeltaTime) override;

private:
    UPROPERTY(VisibleAnywhere)
    UStaticMeshComponent* Mesh;

    UPROPERTY(EditAnywhere)
    FVector Velocity = FVector(100, 0, 0);  // Движение по оси X
};
📌 MovingPlatform.cpp

cpp
Копировать
Редактировать
#include "MovingPlatform.h"

AMovingPlatform::AMovingPlatform()
{
    PrimaryActorTick.bCanEverTick = true;

    Mesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Mesh"));
    RootComponent = Mesh;
}

void AMovingPlatform::BeginPlay()
{
    Super::BeginPlay();
}

void AMovingPlatform::Tick(float DeltaTime)
{
    Super::Tick(DeltaTime);

    FVector NewLocation = GetActorLocation() + (Velocity * DeltaTime);
    SetActorLocation(NewLocation);
}
📌 Что тут происходит?
✅ Создали класс AMovingPlatform, который движется по оси X.
✅ В Tick() каждую секунду добавляем смещение к позиции.
✅ Используем CreateDefaultSubobject для добавления Static Mesh (это объект в сцене).

⚠ Важно: Никогда не создавайте компоненты через new. UE управляет памятью сам.

🔹 2.3. Создание игрового персонажа на C++ (Pawn)
Теперь создадим C++ персонажа с камерой и управлением.

📌 1. Создаем новый C++ класс → Выбираем Pawn → Называем MyPawn.

📌 MyPawn.h

cpp
Копировать
Редактировать
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Pawn.h"
#include "MyPawn.generated.h"

UCLASS()
class MYPROJECT_API AMyPawn : public APawn
{
    GENERATED_BODY()

public:
    AMyPawn();

protected:
    virtual void BeginPlay() override;

public:
    virtual void Tick(float DeltaTime) override;
    virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;

private:
    UPROPERTY(VisibleAnywhere)
    UStaticMeshComponent* Mesh;

    UPROPERTY(VisibleAnywhere)
    class UCameraComponent* Camera;

    void MoveForward(float Value);
};
📌 MyPawn.cpp

cpp
Копировать
Редактировать
#include "MyPawn.h"
#include "Camera/CameraComponent.h"

AMyPawn::AMyPawn()
{
    PrimaryActorTick.bCanEverTick = true;

    Mesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Mesh"));
    RootComponent = Mesh;

    Camera = CreateDefaultSubobject<UCameraComponent>(TEXT("Camera"));
    Camera->SetupAttachment(RootComponent);
}

void AMyPawn::BeginPlay()
{
    Super::BeginPlay();
}

void AMyPawn::Tick(float DeltaTime)
{
    Super::Tick(DeltaTime);
}

void AMyPawn::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
    Super::SetupPlayerInputComponent(PlayerInputComponent);
    PlayerInputComponent->BindAxis("MoveForward", this, &AMyPawn::MoveForward);
}

void AMyPawn::MoveForward(float Value)
{
    if (Value != 0.0f)
    {
        FVector Forward = GetActorForwardVector();
        AddActorWorldOffset(Forward * Value * 100 * GetWorld()->GetDeltaSeconds(), true);
    }
}
📌 Что мы сделали?
✅ Добавили Static Mesh для представления персонажа.
✅ Добавили камеру (CameraComponent).
✅ Прописали ввод: MoveForward() двигает персонажа вперед по W/S.

Теперь в Project Settings → Input создайте Axis Mapping:

MoveForward → W = 1.0, S = -1.0.
🔥 Теперь наш Pawn может двигаться вперед и назад!

🔥 2.4. Разбор работы UObject и Components
UObject – сердце UE5
Все классы UE5 наследуются от UObject или AActor. Это основа для:
✔ Гарбаже-коллектора (GC)
✔ Системы сериализации
✔ Репликации в мультиплеере

💡 ПРАВИЛО:

Если это отдельный объект → это UObject.
Если это что-то на сцене → это AActor.
Если это часть другого объекта → это UComponent.
Пример правильного использования:
🔹 UObject – класс для сохранения данных (например, UInventory).
🔹 AActor – предмет, который можно подобрать (AWeapon).
🔹 UComponent – часть AActor (например, UHealthComponent).

✅ Выводы главы
🔹 Вы изучили Game Framework UE5
🔹 Создали Actor на C++
🔹 Написали Pawn с движением
🔹 Разобрались в UObject и компонентах
