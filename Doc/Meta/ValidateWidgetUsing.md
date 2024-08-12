# ValidateWidgetUsing

Description: 提供一个函数来验证当前属性值是否合法。
Usage: UPROPERTY
Feature: Editor
Group: Scene
Type: bool
LimitedType: 带有MakeEditWidget的FVector，FTransform
Status: Done
Parent item: MakeEditWidget (MakeEditWidget.md)

ValidateWidgetUsing提供一个函数来验证当前属性值是否合法。

- 当前属性要有MakeEditWidget的标记
- 函数的原型是FString MyFunc()，返回非空表示错误信息。

测试代码：

```cpp
	UFUNCTION()
	FString ValidateMyVector()
	{
		if (MyVector_MakeEditWidget_Validate.Length()>100.f)
		{
			return TEXT("Exceed max length:100");
		}
		return TEXT("");
	}

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "EditWidget", meta = (MakeEditWidget, ValidateWidgetUsing = "ValidateMyVector"))
	FVector MyVector_MakeEditWidget_Validate;
```

测试结果：

可见MyVector_MakeEditWidget_Validate长度超过100之后，控件颜色变成红色，并且显示出错误的信息在控件上。

![Untitled](ValidateWidgetUsing/Untitled.png)

原理：

逻辑比较简单。发现有验证函数，就调用验证函数来验证。如果有错误信息，就一起改变最终输出的颜色和显示文字。

```cpp
	static FLegacyEdModeWidgetHelper::FPropertyWidgetInfo CreateWidgetInfo(const TArray<FPropertyWidgetInfoChainElement>& Chain, bool bIsTransform, FProperty* CurrentProp, int32 Index = INDEX_NONE)
	{
		check(CurrentProp);
		FEdMode::FPropertyWidgetInfo WidgetInfo;
		WidgetInfo.PropertyValidationName = FName(*CurrentProp->GetMetaData(FEdMode::MD_ValidateWidgetUsing));
		
		return WidgetInfo;
	}
	
void FLegacyEdModeWidgetHelper::FPropertyWidgetInfo::GetTransformAndColor(UObject* BestSelectedItem, bool bIsSelected, FTransform& OutLocalTransform, FString& OutValidationMessage, FColor& OutDrawColor) const
{
	// Determine the desired color
	if (PropertyValidationName != NAME_None)
	{
		if (UFunction* ValidateFunc = BestSelectedItem->FindFunction(PropertyValidationName))
		{
			BestSelectedItem->ProcessEvent(ValidateFunc, &OutValidationMessage);

			// if we have a negative result, the widget color is red.
			OutDrawColor = OutValidationMessage.IsEmpty() ? OutDrawColor : FColor::Red;
		}
	}
}

void FLegacyEdModeWidgetHelper::DrawHUD(FEditorViewportClient* ViewportClient, FViewport* Viewport, const FSceneView* View, FCanvas* Canvas)
{
	FTransform LocalWidgetTransform;
	FString ValidationMessage;
	FColor WidgetColor;
	WidgetInfo.GetTransformAndColor(BestSelectedItem, bSelected, /*out*/ LocalWidgetTransform, /*out*/ ValidationMessage, /*out*/ WidgetColor);
	
	Canvas->DrawItem(TextItem);
}
```