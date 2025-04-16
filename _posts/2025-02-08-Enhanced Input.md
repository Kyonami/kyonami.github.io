# Enhanced Input

## 개요

구버전의 Input에서 향상된 UE5의 기능.
더 많은 기능을 제공하고, 런타임에서 매핑 정보를 바꾸는 것이 가능함.

## 구성 요소

### Input Action

Digital, 1D, 2D, 3D 등의 이벤트 타입을 지정하는 것이 가능함.

### Input Mapping Context

인풋 액션들을 매핑한 묶음.
캐릭터가 걷다가 수영을 하거나, 탈 것을 타는 등의 환경 변화에 따라 이뤄지는 키 매핑을 런타임에서 편하게 바꿀 수 게 해줌.

### Modifier

Input Action으로 발생하는 Raw Data의 축 변경, 부호 반전 등으로 가공하게 해줌.
마우스로 카메라 이동시 Y축 상하 반전, 한 액션으로 Swizzle, Negate를 통해 상하좌우 구현 가능.

### Trigger

Triggered, Pressed, Released 등 입력 시점 및 동작에 따라 보다 더 다양한 이벤트 발생 가능.

## 방향 입력 예제

| **문자 키** | **방향키** | **원하는 입력 해석** | **필요한 입력 모디파이어**                     |
| ----------- | ---------- | -------------------- | ---------------------------------------------- |
| W           | 위쪽       | 양의 X축             | Swizzle Input Axis Values(YXZ 또는 ZXY)        |
| A           | 왼쪽       | 음의 X축             | Negate                                         |
| S           | 아래쪽     | 음의 Y축             | Negate Swizzle Input Axis Values(YXZ 또는 ZXY) |
| D           | 오른쪽     | 양의 X축             | (없음)                                         |

## 활용 예제

```c++

UCLASS()
class PROJECTMS_API AMSPlayer : public ACharacter
{
private:
	GENERATED_BODY()

public:
	AMSPlayer();

protected:
	virtual void BeginPlay() override;

public:
	virtual void Tick(float DeltaTime) override;

	virtual void SetupPlayerInputComponent(class UInputComponent* InputComponent) override;

private:

	// About Input
	UPROPERTY(EditAnywhere)
	class UInputMappingContext* DefaultContext;

	UPROPERTY(EditAnywhere)
	class UInputAction* MoveAction;

	UPROPERTY(EditAnywhere)
	class UInputAction* JumpAction;

	UPROPERTY(EditAnywhere)
	class UInputAction* CameraAction;

	UPROPERTY(EditAnywhere)
	class UInputAction* InteractAction;


	// About Move
	UPROPERTY(EditAnyWhere,meta=(AllowPrivateAccess = "true"))
	class UCameraComponent* TpsCamera;

	UPROPERTY(EditAnywhere, meta=(AllowPrivateAccess = "true"))
	class USpringArmComponent* TpsSpringArm;

	UPROPERTY(EditAnywhere)
	float Speed = 500.f;

protected:
	void Move(const FInputActionValue& Value);
	void CameraMove(const FInputActionValue& Value);
};

```

```c++

void AMSPlayer::BeginPlay()
{
	Super::BeginPlay();

	if (APlayerController* PlayerController = Cast<APlayerController>(GetController()))
		if (UEnhancedInputLocalPlayerSubsystem* SubSystem =
			ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(PlayerController->GetLocalPlayer()))
			SubSystem->AddMappingContext(DefaultContext, 0);
	
			
	bUseControllerRotationYaw = true;
}

void AMSPlayer::SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);
	if (UEnhancedInputComponent* EnhancedInputComponent = Cast<UEnhancedInputComponent>(PlayerInputComponent))
	{
		EnhancedInputComponent->BindAction(MoveAction, ETriggerEvent::Triggered, this, &AMSPlayer::Move);
		EnhancedInputComponent->BindAction(JumpAction, ETriggerEvent::Triggered, this, &AMSPlayer::Jump);
		EnhancedInputComponent->BindAction(CameraAction, ETriggerEvent::Triggered, this, &AMSPlayer::CameraMove);
		EnhancedInputComponent->BindAction(InteractAction, ETriggerEvent::Triggered, this, &AMSPlayer::Interact);
	}
}

void AMSPlayer::Move(const FInputActionValue& Value)
{
	if (!Controller) return;

	FVector2D MovementVector = Value.Get<FVector2D>();
	FRotationMatrix RotationMatrix = FRotationMatrix(Controller->GetControlRotation());
	FVector ForwardDirection = RotationMatrix.GetUnitAxis(EAxis::X);
	FVector RightDirection = RotationMatrix.GetUnitAxis(EAxis::Y);
 
	ForwardDirection.Z = 0;
	RightDirection.Z = 0;
	
	ForwardDirection.Normalize();
	RightDirection.Normalize();

	AddMovementInput(ForwardDirection, MovementVector.X * Speed);
	AddMovementInput(RightDirection, MovementVector.Y * Speed);
}

void AMSPlayer::CameraMove(const FInputActionValue& Value)
{
	FVector2D Direction(Value.Get<FInputActionValue::Axis2D>());
	AddControllerPitchInput(Direction.Y);
	AddControllerYawInput(Direction.X);
}
```

