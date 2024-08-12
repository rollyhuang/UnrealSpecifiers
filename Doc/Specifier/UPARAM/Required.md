# Required

Description: 指定函数的参数节点必须连接提供一个值
Type: bool
Feature: Blueprint, Parameter
EPropertyFlagsOperation: |=
EPropertyFlags: CPF_RequiredParm (../../Flags/EPropertyFlags/CPF_RequiredParm.md)
Status: Done

如果参数上有提供默认值，该标志依然会忽略默认值，认为还是没提供值。

```cpp
const bool bIsRequiredParam = Param->HasAnyPropertyFlags(CPF_RequiredParm);
	// Don't let the user edit the default value if the parameter is required to be explicit.
	Pin->bDefaultValueIsIgnored |= bIsRequiredParam;
```

测试代码：

```cpp
//PropertyFlags:	CPF_Parm | CPF_ZeroConstructor | CPF_RequiredParm | CPF_NoDestructor | CPF_HasGetValueTypeHash | CPF_NativeAccessSpecifierPublic 
	UFUNCTION(BlueprintCallable)
	FString MyFuncTestParam_RequiredObject(UPARAM(Required) UObject* objValue);

	//(CPP_Default_intValue = 123, ModuleRelativePath = Function/Param/MyFunction_TestParam.h)
	//PropertyFlags:	CPF_Parm | CPF_ZeroConstructor | CPF_RequiredParm | CPF_IsPlainOldData | CPF_NoDestructor | CPF_HasGetValueTypeHash | CPF_NativeAccessSpecifierPublic 
	UFUNCTION(BlueprintCallable)
	FString MyFuncTestParam_RequiredInt(UPARAM(Required) int intValue=123);
```

蓝图节点：

![Untitled](Required/Untitled.png)

如果不连一个节点，编译时会报错：

Pin  Int Value  must be linked to another node (in  My Func Test Param Required Int )
Pin  Obj Value  must be linked to another node (in  My Func Test Param Required Object )