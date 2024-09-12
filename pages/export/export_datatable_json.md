## 导出DataTable为Json

默认枚举是导出枚举名字字符串，如果需要也导出枚举值，修改引擎代码。

```c++
///file:DataTableJSON.cpp

bool TDataTableExporterJSON<CharType>::WriteStructEntry(const void* InRowData, const FProperty* InProperty, const void* InPropertyData)
{
	const FString Identifier = DataTableUtils::GetPropertyExportName(InProperty, DTExportFlags);

	if (const FEnumProperty* EnumProp = CastField<const FEnumProperty>(InProperty))
	{
		const FString PropertyValue = DataTableUtils::GetPropertyValueAsString(EnumProp, (uint8*)InRowData, DTExportFlags);
		JsonWriter->WriteValue(Identifier, PropertyValue);

		//导出枚举值
		UEnum* Enum_object = EnumProp->GetEnum();
		if (Enum_object)
		{
			JsonWriter->WriteValue(Identifier+"_EnumValue", FString::FromInt(Enum_object->GetValueByName(*PropertyValue)));
		}
	}
    ......
}
```