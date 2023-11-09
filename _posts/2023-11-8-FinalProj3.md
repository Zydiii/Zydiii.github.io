---
layout: post
title: "毕设记录（二）"
description: ""
date: 2023-5-15
feature_image: images/2023.11.8/0.png
tags: [毕设]
---

<!--more-->

## Adding Shading Model

### 新增 MaterialShadingModel

- **EngineTypes.h** 添加 ShadingModel 的枚举类型，需要在`MSM_NUM` 之前进行添加，否则会报 `check(InShadingModel < MSM_NUM)` 检查
  
```C++
UENUM()
enum EMaterialShadingModel
{
	MSM_Unlit					UMETA(DisplayName="Unlit"),
	MSM_DefaultLit				UMETA(DisplayName="Default Lit"),
	MSM_Subsurface				UMETA(DisplayName="Subsurface"),
	MSM_PreintegratedSkin		UMETA(DisplayName="Preintegrated Skin"),
	MSM_ClearCoat				UMETA(DisplayName="Clear Coat"),
	MSM_SubsurfaceProfile		UMETA(DisplayName="Subsurface Profile"),
	MSM_TwoSidedFoliage			UMETA(DisplayName="Two Sided Foliage"),
	MSM_Hair					UMETA(DisplayName="Hair"),
	MSM_Cloth					UMETA(DisplayName="Cloth"),
	MSM_Eye						UMETA(DisplayName="Eye"),
	MSM_SingleLayerWater		UMETA(DisplayName="SingleLayerWater"),
	MSM_ThinTranslucent			UMETA(DisplayName="Thin Translucent"),
	MSM_Strata					UMETA(DisplayName="Strata", Hidden),
	// Our custom shading model
	MSM_CS_Default				UMETA(DisplayName = "Celshading default"),
	MSM_CS_Subsurface			UMETA(DisplayName = "Celshading subsurface"),
	MSM_CS_Skin					UMETA(DisplayName = "Celshading skin"),
	MSM_Infrared_Default		UMETA(DisplayName = "Infrared Default"),

	/** Number of unique shading models. */
	MSM_NUM						UMETA(Hidden),
	/** Shading model will be determined by the Material Expression Graph,
		by utilizing the 'Shading Model' MaterialAttribute output pin. */
	MSM_FromMaterialExpression	UMETA(DisplayName="From Material Expression"),
	MSM_MAX
};
```

- **HLSLMaterialTranslator.cpp** 里添加 HLSL 中的 shading model 宏定义声明

```C++
if (ShadingModels.HasShadingModel(MSM_CS_Default))
{
	OutEnvironment.SetDefine(TEXT("MATERIAL_SHADINGMODEL_CS_DEFAULT"), TEXT("1"));
	NumSetMaterials++;
}

if (ShadingModels.HasShadingModel(MSM_CS_Subsurface))
{
	OutEnvironment.SetDefine(TEXT("MATERIAL_SHADINGMODEL_CS_SUBSURFACE"), TEXT("1"));
	NumSetMaterials++;
}

if (ShadingModels.HasShadingModel(MSM_CS_Skin))
{
	OutEnvironment.SetDefine(TEXT("MATERIAL_SHADINGMODEL_CS_SKIN"), TEXT("1"));
	NumSetMaterials++;
}

if (ShadingModels.HasShadingModel(MSM_Infrared_Default))
{
	OutEnvironment.SetDefine(TEXT("MATERIAL_SHADINGMODEL_Infrared_Default"), TEXT("1"));
	NumSetMaterials++;
}
```

## 小结

## References

- [New shading models and changing the GBuffer](https://dev.epicgames.com/community/learning/tutorials/2R5x/unreal-engine-new-shading-models-and-changing-the-gbuffer?locale=ru-ru)
- [Adding a new Shading Model in UE5](https://zhuanlan.zhihu.com/p/553585780)
