---
layout: post
title: "毕设记录（四）"
description: ""
date: 2023-11-8
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
	OutEnvironment.SetDefine(TEXT("MATERIAL_SHADINGMODEL_INFRARED_DEFAULT"), TEXT("1"));
	NumSetMaterials++;
}
```

- **ShaderMaterial.h** 中为 shadingmodel 的 define 做映射

```GLSL
uint8 MATERIAL_SHADINGMODEL_CS_DEFAULT : 1;
uint8 MATERIAL_SHADINGMODEL_CS_SUBSURFACE : 1;
uint8 MATERIAL_SHADINGMODEL_CS_SKIN : 1;
uint8 MATERIAL_SHADINGMODEL_INFRARED_DEFAULT : 1;
```

### 开启 CustomData 接口

- **Material.cpp** 激活输出节点 `CustomData0` 和 `CustomData1`

```C++
case MP_CustomData0:
	Active = ShadingModels.HasAnyShadingModel({ MSM_ClearCoat, MSM_Hair, MSM_Cloth, MSM_Eye, MSM_SubsurfaceProfile, MSM_CS_Default, MSM_CS_Subsurface, MSM_CS_Skin, MSM_Infrared_Default });
	break;
case MP_CustomData1:
	Active = ShadingModels.HasAnyShadingModel({ MSM_ClearCoat, MSM_Eye, MSM_CS_Default, MSM_CS_Subsurface, MSM_CS_Skin, MSM_Infrared_Default });
	break;
```

- **MaterialShared.cpp** 设置 `CustomData0` 和 `CustomData1` 的接口显示名字

```C++
case MP_CustomData0:	
	CustomPinNames.Add({ MSM_ClearCoat, "Clear Coat" });
	CustomPinNames.Add({MSM_Hair, "Backlit"});
	CustomPinNames.Add({MSM_Cloth, "Cloth"});
	CustomPinNames.Add({MSM_Eye, "Iris Mask"});
	CustomPinNames.Add({MSM_SubsurfaceProfile, "Curvature" });
	CustomPinNames.Add({ MSM_CS_Default, "Temperature" });
	CustomPinNames.Add({ MSM_CS_Subsurface, "Temperature" });
	CustomPinNames.Add({ MSM_CS_Skin, "Temperature" });
	CustomPinNames.Add({ MSM_Infrared_Default, "Temperature" });
	return FText::FromString(GetPinNameFromShadingModelField(Material->GetShadingModels(), CustomPinNames, "Custom Data 0"));
case MP_CustomData1:
	CustomPinNames.Add({ MSM_ClearCoat, "Clear Coat Roughness" });
	CustomPinNames.Add({MSM_Eye, "Iris Distance"});
	CustomPinNames.Add({ MSM_CS_Default, "Emissivity" });
	CustomPinNames.Add({ MSM_CS_Subsurface, "Emissivity" });
	CustomPinNames.Add({ MSM_CS_Skin, "Emissivity" });
	CustomPinNames.Add({ MSM_Infrared_Default, "Emissivity" });
	return FText::FromString(GetPinNameFromShadingModelField(Material->GetShadingModels(), CustomPinNames, "Custom Data 1"));
```

- **ShaderMaterialDerivedHelpers.cpp** 设置允许写入 customdata 到 buffer 中 ？

```C++
Dst.WRITES_CUSTOMDATA_TO_GBUFFER = (Dst.USES_GBUFFER && (Mat.MATERIAL_SHADINGMODEL_SUBSURFACE || Mat.MATERIAL_SHADINGMODEL_PREINTEGRATED_SKIN || Mat.MATERIAL_SHADINGMODEL_SUBSURFACE_PROFILE || Mat.MATERIAL_SHADINGMODEL_CLEAR_COAT || Mat.MATERIAL_SHADINGMODEL_TWOSIDED_FOLIAGE || Mat.MATERIAL_SHADINGMODEL_HAIR || Mat.MATERIAL_SHADINGMODEL_CLOTH || Mat.MATERIAL_SHADINGMODEL_EYE || Mat.MATERIAL_SHADINGMODEL_CS_DEFAULT || Mat.MATERIAL_SHADINGMODEL_CS_SKIN || Mat.MATERIAL_SHADINGMODEL_CS_SUBSURFACE || Mat.MATERIAL_SHADINGMODEL_INFRARED_DEFAULT));
```

### 设置 GBuffer Shading Model ID

- **ShadingCommon.ush** 添加 Shading model 的定义，并设置 shading model 在 gbuffer:shading model 的颜色

```HLSL
// SHADINGMODELID_* occupy the 4 low bits of an 8bit channel and SKIP_* occupy the 4 high bits
#define SHADINGMODELID_UNLIT				0
#define SHADINGMODELID_DEFAULT_LIT			1
#define SHADINGMODELID_SUBSURFACE			2
#define SHADINGMODELID_PREINTEGRATED_SKIN	3
#define SHADINGMODELID_CLEAR_COAT			4
#define SHADINGMODELID_SUBSURFACE_PROFILE	5
#define SHADINGMODELID_TWOSIDED_FOLIAGE		6
#define SHADINGMODELID_HAIR					7
#define SHADINGMODELID_CLOTH				8
#define SHADINGMODELID_EYE					9
#define SHADINGMODELID_SINGLELAYERWATER		10
#define SHADINGMODELID_THIN_TRANSLUCENT		11
#define SHADINGMODELID_STRATA				12		// Temporary while we convert everything to Strata
#define SHADINGMODELID_CS_DEFAULT			13		// Celshading Lucas: create material celshading defines for shaders
#define SHADINGMODELID_CS_SUBSURFACE		14
#define SHADINGMODELID_CS_SKIN				15
#define SHADINGMODELID_INFRARED_DEFAULT		16
#define SHADINGMODELID_NUM					17
#define SHADINGMODELID_MASK					0xF		// 4 bits reserved for ShadingModelID

// for debugging and to visualize
float3 GetShadingModelColor(uint ShadingModelID)
{
	// TODO: PS4 doesn't optimize out correctly the switch(), so it thinks it needs all the Samplers even if they get compiled out
	//	This will get fixed after launch per Sony...
#if PS4_PROFILE
		 if (ShadingModelID == SHADINGMODELID_UNLIT) return float3(0.1f, 0.1f, 0.2f); // Dark Blue
	else if (ShadingModelID == SHADINGMODELID_DEFAULT_LIT) return float3(0.1f, 1.0f, 0.1f); // Green
	else if (ShadingModelID == SHADINGMODELID_SUBSURFACE) return float3(1.0f, 0.1f, 0.1f); // Red
	else if (ShadingModelID == SHADINGMODELID_PREINTEGRATED_SKIN) return float3(0.6f, 0.4f, 0.1f); // Brown
	else if (ShadingModelID == SHADINGMODELID_CLEAR_COAT) return float3(0.1f, 0.4f, 0.4f); 
	else if (ShadingModelID == SHADINGMODELID_SUBSURFACE_PROFILE) return float3(0.2f, 0.6f, 0.5f); // Cyan
	else if (ShadingModelID == SHADINGMODELID_TWOSIDED_FOLIAGE) return float3(0.2f, 0.2f, 0.8f); // Blue
	else if (ShadingModelID == SHADINGMODELID_HAIR) return float3(0.6f, 0.1f, 0.5f);
	else if (ShadingModelID == SHADINGMODELID_CLOTH) return float3(0.7f, 1.0f, 1.0f); 
	else if (ShadingModelID == SHADINGMODELID_EYE) return float3(0.3f, 1.0f, 1.0f); 
	else if (ShadingModelID == SHADINGMODELID_SINGLELAYERWATER) return float3(0.5f, 0.5f, 1.0f);
	else if (ShadingModelID == SHADINGMODELID_THIN_TRANSLUCENT) return float3(1.0f, 0.8f, 0.3f);
	else if (ShadingModelID == SHADINGMODELID_STRATA) return float3(1.0f, 1.0f, 0.0f);
	else if (ShadingModelID == SHADINGMODELID_CS_DEFAULT) return float3(0.32f, 0.11f, 0.14f); // dark red Celshading Lucas: set color for shading model debug view
	else if (ShadingModelID == SHADINGMODELID_CS_SUBSURFACE) return float3(0.50f, 0.23f, 0.52f); // purple
	else if (ShadingModelID == SHADINGMODELID_CS_SKIN) return float3(0.33f, 0.59f, 0.76f); // blue
	else if (ShadingModelID == SHADINGMODELID_INFRARED_DEFAULT) return float3(0.76f, 0.35f, 0.39f); // red
	else return float3(1.0f, 1.0f, 1.0f); // White
#else
	switch(ShadingModelID)
	{
		case SHADINGMODELID_UNLIT: return float3(0.1f, 0.1f, 0.2f); // Dark Blue
		case SHADINGMODELID_DEFAULT_LIT: return float3(0.1f, 1.0f, 0.1f); // Green
		case SHADINGMODELID_SUBSURFACE: return float3(1.0f, 0.1f, 0.1f); // Red
		case SHADINGMODELID_PREINTEGRATED_SKIN: return float3(0.6f, 0.4f, 0.1f); // Brown
		case SHADINGMODELID_CLEAR_COAT: return float3(0.1f, 0.4f, 0.4f); // Brown
		case SHADINGMODELID_SUBSURFACE_PROFILE: return float3(0.2f, 0.6f, 0.5f); // Cyan
		case SHADINGMODELID_TWOSIDED_FOLIAGE: return float3(0.2f, 0.2f, 0.8f); // Cyan
		case SHADINGMODELID_HAIR: return float3(0.6f, 0.1f, 0.5f);
		case SHADINGMODELID_CLOTH: return float3(0.7f, 1.0f, 1.0f);
		case SHADINGMODELID_EYE: return float3(0.3f, 1.0f, 1.0f);
		case SHADINGMODELID_SINGLELAYERWATER: return float3(0.5f, 0.5f, 1.0f);
		case SHADINGMODELID_THIN_TRANSLUCENT: return float3(1.0f, 0.8f, 0.3f);
		case SHADINGMODELID_STRATA: return float3(1.0f, 1.0f, 0.0f);
		case SHADINGMODELID_CS_DEFAULT: return float3(0.32f, 0.11f, 0.14f); // dark red Celshading Lucas: set color for shading model debug view
		case SHADINGMODELID_CS_SUBSURFACE: return float3(0.50f, 0.23f, 0.52f); // purple
		case SHADINGMODELID_CS_SKIN: return float3(0.33f, 0.59f, 0.76f); // blue
		case SHADINGMODELID_INFRARED_DEFAULT: return float3(0.76f, 0.35f, 0.39f); // red
		default: return float3(1.0f, 1.0f, 1.0f); // White
	}
#endif
}
```

- **BasePassPixelShader.usf** 声明 shading model ？

```HLSL
#if MATERIAL_SHADINGMODEL_INFRARED_DEFAULT
	uint ShadingModel = SHADINGMODELID_INFRARED_DEFAULT;
#else
	uint ShadingModel = GetMaterialShadingModel(PixelMaterialInputs);
#endif
```

### 将 CustomData 写入 GBuffer

- **ShadingModelsMaterial.ush** 控制 shading model 写入 FGbufferData 的数据，这里使用GetMaterialCustomData0来获取CustomData0的输入，并将其存放到GBuffer.CustomData中

```HLSL
#if MATERIAL_SHADINGMODEL_INFRARED_DEFAULT
	else if (ShadingModel == SHADINGMODELID_INFRARED_DEFAULT)
	{
		GBuffer.ShadingModelID = SHADINGMODELID_INFRARED_DEFAULT;
		GBuffer.CustomData.x = GetMaterialCustomData0(MaterialParameters);
		GBuffer.CustomData.y = GetMaterialCustomData1(MaterialParameters);
		GBuffer.CustomData.z = GetCelShadingSelection0(MaterialParameters);
	}
#endif
```

- **BasePassCommon.ush** 开放写入 customdata to gbuffer 权限

```HLSL
#define WRITES_CUSTOMDATA_TO_GBUFFER		(USES_GBUFFER && (MATERIAL_SHADINGMODEL_SUBSURFACE || MATERIAL_SHADINGMODEL_PREINTEGRATED_SKIN || MATERIAL_SHADINGMODEL_SUBSURFACE_PROFILE || MATERIAL_SHADINGMODEL_CLEAR_COAT || MATERIAL_SHADINGMODEL_TWOSIDED_FOLIAGE || MATERIAL_SHADINGMODEL_HAIR || MATERIAL_SHADINGMODEL_CLOTH || MATERIAL_SHADINGMODEL_EYE || MATERIAL_SHADINGMODEL_INFRARED_DEFAULT || MATERIAL_SHADINGMODEL_CS_DEFAULT || MATERIAL_SHADINGMODEL_CS_SUBSURFACE))
```

- **DeferredShadingCommon.ush** 设置 shading model 是否拥有 custom data gbuffer

```HLSL
bool HasCustomGBufferData(int ShadingModelID)
{
	return ShadingModelID == SHADINGMODELID_SUBSURFACE
		|| ShadingModelID == SHADINGMODELID_PREINTEGRATED_SKIN
		|| ShadingModelID == SHADINGMODELID_CLEAR_COAT
		|| ShadingModelID == SHADINGMODELID_SUBSURFACE_PROFILE
		|| ShadingModelID == SHADINGMODELID_TWOSIDED_FOLIAGE
		|| ShadingModelID == SHADINGMODELID_HAIR
		|| ShadingModelID == SHADINGMODELID_CLOTH
		|| ShadingModelID == SHADINGMODELID_EYE
		|| ShadingModelID == SHADINGMODELID_INFRARED_DEFAULT
		|| ShadingModelID == SHADINGMODELID_CS_DEFAULT
		|| ShadingModelID == SHADINGMODELID_CS_SUBSURFACE;
}
```

### C++ 绑定 Shader

- **ShaderGenerationUtil.cpp** 声明 shader compile，并设置对应的 gbuffer slot

```C++
//Celshading Lucas: Fetch celshading model bools
FETCH_COMPILE_BOOL(MATERIAL_SHADINGMODEL_CS_DEFAULT);
FETCH_COMPILE_BOOL(MATERIAL_SHADINGMODEL_CS_SUBSURFACE);
FETCH_COMPILE_BOOL(MATERIAL_SHADINGMODEL_INFRARED_DEFAULT);

//Celshading Lucas: tell which gbuffer slot we're writting in.
	if (Mat.MATERIAL_SHADINGMODEL_CS_DEFAULT)
	{
		//#IF USE_NEW_GBUFFER
		SetStandardGBufferSlots(Slots, bWriteEmissive, bHasTangent, bHasVelocity, bHasStaticLighting, bIsStrataMaterial);
		Slots[GBS_Celshading] = true;
		//#ELSE
		SetStandardGBufferSlots(Slots, bWriteEmissive, bHasTangent, bHasVelocity, bHasStaticLighting, bIsStrataMaterial);
		Slots[GBS_CustomData] = bUseCustomData;
		//#ENDIF
	}

	if (Mat.MATERIAL_SHADINGMODEL_CS_SUBSURFACE)
	{
		SetStandardGBufferSlots(Slots, bWriteEmissive, bHasTangent, bHasVelocity, bHasStaticLighting, bIsStrataMaterial);
		Slots[GBS_CustomData] = bUseCustomData;
		//#IF USE_NEW_GBUFFER
		Slots[GBS_Celshading] = true;
		//#ENDIF
	}

	if (Mat.MATERIAL_SHADINGMODEL_INFRARED_DEFAULT)
	{
		SetStandardGBufferSlots(Slots, bWriteEmissive, bHasTangent, bHasVelocity, bHasStaticLighting, bIsStrataMaterial);
		Slots[GBS_CustomData] = bUseCustomData;
		//#IF USE_NEW_GBUFFER
		Slots[GBS_Celshading] = true;
		//#ENDIF
	}
```

- **Definitions.usf** 声明 shading model

```HLSL
#ifndef MATERIAL_SHADINGMODEL_INFRARED_DEFAULT
#define MATERIAL_SHADINGMODEL_INFRARED_DEFAULT		0
#endif
```

### 光照修改

- **ClusteredDeferredShadingPixelShader.usf** 添加 shading model

```GLSL
GET_LIGHT_GRID_LOCAL_LIGHTING_SINGLE_SM(SHADINGMODELID_INFRARED_DEFAULT,	PixelShadingModelID, CompositedLighting, ScreenUV, CulledLightGridData, Dither, FirstNonSimpleLightIndex);
GET_LIGHT_GRID_LOCAL_LIGHTING_SINGLE_SM(SHADINGMODELID_CS_DEFAULT,			PixelShadingModelID, CompositedLighting, ScreenUV, CulledLightGridData, Dither, FirstNonSimpleLightIndex);
GET_LIGHT_GRID_LOCAL_LIGHTING_SINGLE_SM(SHADINGMODELID_CS_SUBSURFACE,		PixelShadingModelID, CompositedLighting, ScreenUV, CulledLightGridData, Dither, FirstNonSimpleLightIndex);	
```

- **ShadingModels.ush** 在 `IntegrateBxDF` 添加对应的着色模型，改了但是不起作用。。

```GLSL
FDirectLighting IntegrateBxDF( FGBufferData GBuffer, half3 N, half3 V, half3 L, float Falloff, half NoL, FAreaLight AreaLight, FShadowTerms Shadow )
{
	switch( GBuffer.ShadingModelID )
	{
		case SHADINGMODELID_DEFAULT_LIT:
		case SHADINGMODELID_SINGLELAYERWATER:
		case SHADINGMODELID_THIN_TRANSLUCENT:
			return DefaultLitBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
		case SHADINGMODELID_SUBSURFACE:
			return SubsurfaceBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
		case SHADINGMODELID_PREINTEGRATED_SKIN:
			return PreintegratedSkinBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
		case SHADINGMODELID_CLEAR_COAT:
			return ClearCoatBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
		case SHADINGMODELID_SUBSURFACE_PROFILE:
			return SubsurfaceProfileBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
		case SHADINGMODELID_TWOSIDED_FOLIAGE:
			return TwoSidedBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
		case SHADINGMODELID_HAIR:
			return HairBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
		case SHADINGMODELID_CLOTH:
			return ClothBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
		case SHADINGMODELID_EYE:
			return EyeBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
		case SHADINGMODELID_CS_DEFAULT:
			//#IF USE_NEW_GBUFFER
			return CelshadingDefaultLitBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow, GBuffer.Celshading.x);
			//#ELSE
			//return CelshadingDefaultLitBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow, GBuffer.CustomData.a);
			//#ENDIF
		case SHADINGMODELID_CS_SUBSURFACE:
			return CelshadingSubsurfaceBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
		case SHADINGMODELID_INFRARED_DEFAULT:
			return InfraredDefaultBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
		default:
			return (FDirectLighting)1.0f;
		//default:
			//return (FDirectLighting)0;
	}
}

FDirectLighting InfraredDefaultBxDF( FGBufferData GBuffer, half3 N, half3 V, half3 L, float Falloff, half NoL, FAreaLight AreaLight, FShadowTerms Shadow )
{
	FDirectLighting Lighting;
	Lighting.Diffuse = 0;
	Lighting.Specular = 0;
	Lighting.Transmission = 0;

	return Lighting;
}
```

### Reading GBuffer Data

- 在 PostPorcess 的 Custom Node 中直接访问

#if SCENE_TEXTURES_DISABLED
    return float4(0, 0, 0, 0);
#else
    FScreenSpaceData ScreenSpaceData = GetScreenSpaceData(GetDefaultSceneTextureUV(Parameters, 15), false);
    return ScreenSpaceData.GBuffer.Celshading.rgba;
#endif

GetDefaultSceneTextureUV(Parameters, 15)

#if SCENE_TEXTURES_DISABLED
    return float3(0, 0, 0);
#else
    FScreenSpaceData ScreenSpaceData = GetScreenSpaceData(GetDefaultSceneTextureUV(Parameters, 8), false);
    return ScreenSpaceData.GBuffer.CustomData.rgb;
#endif

#if SCENE_TEXTURES_DISABLED
    return float3(0, 0, 0);
#else
    FScreenSpaceData ScreenSpaceData = GetScreenSpaceData(GetDefaultSceneTextureUV(Parameters, 8), false);
    return ScreenSpaceData.GBuffer.Celshading.rgb;
#endif

## Rendering Pipeline

- 收集阶段 & InitView：StaticMesh 缓存好的 Command 和 Mesh，DynamicMesh 需要构建 Command，将 Static 和 Dynamic 都交给 SetupMeshPass，生成 MeshDrawCommand
- MeshDrawCommand 到 RHI 阶段：EMeshPass 有多种 Pass 类型
  - 使用 FMeshPassProcessor 选择绘制时使用的 shader 并搜集这个 pass 绑定的顶点工厂、材质等，MeshBatch 可以创建多个 MeshDrawCommand
  - 初始化的时候，会往 MeshPassProcessor 里填充 BatchMesh 渲染单元，调用 AddMeshBatch FDepthPassMeshProcessor::TryAddMeshBatch()，会进行 Passfilter 和 SelectShader，应该就是检查是否是透明物体、是否该在这个 pass 渲染。然后设置好渲染需要的数据比如 Shader 等之后通过 BuildMeshDrawCommands 创建 command 并 FinalizeCommand 添加 command
  - MeshDrawCommand 描述了一次绘制所需要的资源，shaderbinding、vertexbuffer、indexbuffer、graphicpipelinestate 等
  - PrePass：把非透明和Mask物体进行深度排序渲染到 DepthMap，通过 GraphBuilder.AddPass View.ParallelMeshDrawCommandPasses[EMeshPass::DepthPass].DispatchDraw 来创建并行任务，生成对应的命令
- 往场景中添加一个物体时，会调用 AddPrimitive，创建场景代理，更新 GPUScene，然后 CacheMeshDrawCommands。然后遍历所有 Pass 类型创建对应的 PassProcessor 然后 AddMeshBatch，这个 Processor 会创建 MeshDrawCommands 并将其存入 Cache 中，完成创建之后便销毁这个 Proceesor，完成了对 commands 的添加
- InitViews 准备这一帧所需要的绘制数据，ComputeViewVisibility 用于最后的绘制数据的剔除准备工作
- Static 数据是预生成到 Cache 中的，Dynamic 的 MeshDrawCommand 是每帧重新生成的。准备好剔除后的渲染数据之后，需要在 ComputeAndMarkRelevanceForViewParallel 中将 MeshCommand 取出来。动态物体使用 GenerateDynamicMeshDrawCommands 生成，生成后加到每个 Mesh
- DispatchPassSetup 负责为每个 pass 管理 MeshDrawCommands，为绘制每个 pass 做准备
- Pass 从 View.ParallelMeshDrawCommandPasses[EMeshPass::CustomDepth] 里面取出对应的 Commands
- 要添加自己的 Pass，首先构造出需要的 MeshDrawCommand，然后调用 MeshDrawCommand 进行绘制
  - 添加 MeshPass 枚举
  - 添加对应的 Processor，重点是重载 AddMeshBatch，提交符合要求的 MeshBatch
  - 添加相应的 MeshMaterialShader 绑定 VS 和 PS
  - 在 DeferredShadingSceneRenderer 中添加该 Pass 的入口函数
  - 添加相关性，分别绑定 static 和 dynamic mesh，然后在 Render() 中调用添加的 RenderPass 入口函数

recompileshaders changed

## Add Custom Pass

参考的这个 https://zhuanlan.zhihu.com/p/613772622

### 添加新的 Pass 类型

- **MeshPassProcessor.h**
  - EMeshPass 添加 CustomPass，NumBits 改成 6
  - GetMeshPassName 添加对应的 case EMeshPass::CustomPass: return TEXT("CustomPass");
  - 因为数量变多了，需要修改一些检查 static_assert(EMeshPass::Num == 29 + 4, "Need to update switch(MeshPass) after changing EMeshPass"); FMeshPassMask，
- **MicrosoftPlatformMath.h** 加了 CountTrailingZerosTmp
- **PSOPrecache.h** 改 MaxPSOCollectorCount 33

### 新建 shader

- **Engine/Shaders/Private/CustomPassShader.usf**
  - 这个还在改比较乱，目前只有 MainVS 和 MainPS 有用

### 新建 VS、PS、MeshProcessor、注册 pass

- **Engine\Source\Runtime\Renderer\Private** 
  - 加 CustomRendering.h 和 CustomRendering.cpp

### 添加 RenderPass 入口

- **DeferredShadingRenderer.h** 加上 RenderCustomPass()
- **DeferredShadingRenderer.cpp** 调用 RenderCustomPass()

### 关联 Pass 可见性

- **SceneVisibility.cpp** 加上 EMeshPass::CustomPass

## 在fork的项目里同步别人新增分支的方法

```shell
# 1.将项目B clone 到本地
git clone -b master 项目B的git地址

# 2.将项目A的git地址，添加至本地的remote
git remote add upstream 项目A的git地址

# 3.在本地新建一个分支，该分支的名称最好与项目A中新增的那个分支的名称相同以便区分
git checkout -b 新分支名称

# 4.从项目A中将新分支的内容 pull 到本地
git pull upstream 新分支名称

# 5.将 pull 下来的分支 push 到项目B 中去
git push origin 新分支名称
```

## Debug

### GetMaterialCustomData0 

- 与 MaterialProxy 有关系，之前是用的 `const FMaterialRenderProxy& DefualtProxy1 = *UMaterial::GetDefaultMaterial(MD_Surface)->GetRenderProxy();`，推测这是获得的默认材质，所以返回值都是默认值，需要用 MeshBatch.MaterialRenderProxy

```C++
void FCustomPassMeshProcessor::AddMeshBatch(const FMeshBatch& RESTRICT MeshBatch, uint64 BatchElementMask, const FPrimitiveSceneProxy* RESTRICT PrimitiveSceneProxy, int32 StaticMeshId)
{
	const FMaterialRenderProxy* FallBackMaterialRenderProxyPtr = nullptr;
	const FMaterial& Material = MeshBatch.MaterialRenderProxy->GetMaterialWithFallback(Scene->GetFeatureLevel(), FallBackMaterialRenderProxyPtr);
	const FMeshDrawingPolicyOverrideSettings OverrideSettings = ComputeMeshOverrideSettings(MeshBatch);
	const ERasterizerFillMode MeshFillMode = ComputeMeshFillMode(Material, OverrideSettings);
	const ERasterizerCullMode MeshCullMode = ComputeMeshCullMode(Material, OverrideSettings);
	const EBlendMode BlendMode = Material.GetBlendMode();
	const bool bIsTranslucent = IsTranslucentBlendMode(BlendMode);

	if (Material.GetShadingModels().HasShadingModel(MSM_Infrared_Default))
	{
		// 只有不透明物体
		// const FMaterialRenderProxy* DefualtProxy1 = MeshBatch.MaterialRenderProxy;
		const FMaterialRenderProxy* DefualtProxy1 = UMaterial::GetDefaultMaterial(MD_Surface)->GetRenderProxy();
		const FMaterial& DefaltMaterial1 = DefualtProxy1->GetIncompleteMaterialWithFallback(Scene->GetFeatureLevel());

		Process(
				MeshBatch,
				BatchElementMask,
				StaticMeshId,
				PrimitiveSceneProxy,
				*DefualtProxy1,
				DefaltMaterial1,
				MeshFillMode,
				MeshCullMode
		);
	}
}
```

### 透明物体

- v0 只有不透明物体，而且行为和 BasePass 一致？？
```C++
void FCustomPassMeshProcessor::AddMeshBatch(const FMeshBatch& RESTRICT MeshBatch, uint64 BatchElementMask, const FPrimitiveSceneProxy* RESTRICT PrimitiveSceneProxy, int32 StaticMeshId)
{
	const FMaterialRenderProxy* FallBackMaterialRenderProxyPtr = nullptr;
	const FMaterial& Material = MeshBatch.MaterialRenderProxy->GetMaterialWithFallback(Scene->GetFeatureLevel(), FallBackMaterialRenderProxyPtr);
	const FMeshDrawingPolicyOverrideSettings OverrideSettings = ComputeMeshOverrideSettings(MeshBatch);
	const ERasterizerFillMode MeshFillMode = ComputeMeshFillMode(Material, OverrideSettings);
	const ERasterizerCullMode MeshCullMode = ComputeMeshCullMode(Material, OverrideSettings);
	const EBlendMode BlendMode = Material.GetBlendMode();
	const bool bIsTranslucent = IsTranslucentBlendMode(BlendMode);

	if (Material.GetShadingModels().HasShadingModel(MSM_Infrared_Default))
	{
		const FMaterialRenderProxy* DefualtProxy1 = MeshBatch.MaterialRenderProxy;
		const FMaterial& DefaltMaterial1 = DefualtProxy1->GetIncompleteMaterialWithFallback(Scene->GetFeatureLevel());

		Process(
				MeshBatch,
				BatchElementMask,
				StaticMeshId,
				PrimitiveSceneProxy,
				*DefualtProxy1,
				DefaltMaterial1,
				MeshFillMode,
				MeshCullMode
		);
	}
}
```

- 在实测中发现，虽然添加了所有物体，但是半透明物体并没有被绘制，其中一个影响比较大的句子是 `const FMaterial& Material = MeshBatch.MaterialRenderProxy->GetMaterialWithFallback(Scene->GetFeatureLevel(), FallBackMaterialRenderProxyPtr); if (Material.GetShadingModels().HasShadingModel(MSM_Infrared_Default))`，把 Process() 写在这个判断中那么半透明物体全都没有被添加，推测可能是这个拿到的 Material 并不是当前的目标对象的，至少对于半透明物体来说是这样，拿到的这个没有 shading model 之类的信息，所以 if 条件没通过，但是不透明物体又能过，不是很明白这个原理。但这里其实也比较奇怪，因为一开始的默认版本是有所有物体的，用的同一个 material 来做的判断
- 上面这个解决了部分半透明物体不绘制的问题，但是实测中发现半透明物体有没有被添加上还和自身的 Opacity 有关系，如果 Opacity 比较小就完全没有被绘制，同时还存在绘制的物体颜色和预期不符合，都是 1，有点奇怪，难道会影响什么排序之类的吗，不太明白
- v1

```C++
void FCustomPassMeshProcessor::AddMeshBatch(const FMeshBatch& RESTRICT MeshBatch, uint64 BatchElementMask, const FPrimitiveSceneProxy* RESTRICT PrimitiveSceneProxy, int32 StaticMeshId)
{
	const FMaterialRenderProxy* MaterialRenderProxy = MeshBatch.MaterialRenderProxy;
	while (MaterialRenderProxy)
	{
		const FMaterial* Material1 = MaterialRenderProxy->GetMaterialNoFallback(FeatureLevel);

		// if (Material1) // 取消判断 Material1->GetShadingModels().HasShadingModel(MSM_Infrared_Default) 低透明度的也没有
		if (Material1 && Material1->GetShadingModels().HasShadingModel(MSM_Infrared_Default))
		{
			if (TryAddMeshBatch(MeshBatch, BatchElementMask, PrimitiveSceneProxy, StaticMeshId, *MaterialRenderProxy, *Material1, *Material1))
			{
				break;
			}
		}

		MaterialRenderProxy = MaterialRenderProxy->GetFallback(FeatureLevel);
	}
}
void FCustomPassMeshProcessor::AddMeshBatch(const FMeshBatch& RESTRICT MeshBatch, uint64 BatchElementMask, const FPrimitiveSceneProxy* RESTRICT PrimitiveSceneProxy, int32 StaticMeshId)
{
	const FMaterialRenderProxy* MaterialRenderProxy = MeshBatch.MaterialRenderProxy;
	while (MaterialRenderProxy)
	{
		const FMaterial* Material1 = &MaterialRenderProxy->GetIncompleteMaterialWithFallback(Scene->GetFeatureLevel());

		//if (Material && Material->GetRenderingThreadShaderMap() && Material->GetShadingModels().HasShadingModel(MSM_Infrared_Default))
		if (Material1 && Material1->GetShadingModels().HasShadingModel(MSM_Infrared_Default))
		{
			if (TryAddMeshBatch(MeshBatch, BatchElementMask, PrimitiveSceneProxy, StaticMeshId, *MaterialRenderProxy, *Material1, *Material1))
			{
				break;
			}
		}

		MaterialRenderProxy = MaterialRenderProxy->GetFallback(FeatureLevel);
	}
}
```

- 好吧，问题出在，shader 对 opacitymask 做了 clip！ue 下默认的 opacitymask 是 0.333，所以才会出现比较小的不透明度的物体不绘制，因为在 PS 端就被 clip 了！找了我整整一天，一直以为是 CPU 那边的问题，结果通过在 VS 对 depth 做了更改，发现所有物体都没有通过深度测试，才意识到 AddMeshBatch 那边没少（不过一开始的那个判断也还是有问题，不能提前判断？待验证）
- 实测中还是遇到问题了，发现 get shader 始终是失败的，半透明物体并没有被加到我的 pass 中，但是比较奇怪的点在于如果我重启了，就会被添加进去，然后最终 build 出来的版本还是没有，所有半透明物体都没有加到 pass 中，好困惑。目前解决方案是用 customdepthpass 直接写入数据了，但比较看起来明明没啥区别，等之后有空再研究吧

### depth fade

- 实测中发现用了 depth fade 的材质一用就崩溃，通过在 custom pass 的 passparameters 添加 uniform SceneTexture 似乎解决了，应该是这个节点要引用 scenetextures 的一些数据，同时 custompass 还不能放得太靠前，因为 textures 可能还没有构建出来，但为什么只和我这个 pass 有关呢，而且我的 pass 明明也没有引用相关的东西，不太理解

### Niagara

- Niagara 的 Sprite 和 Translucent Mesh Render 似乎不太对，拿到的 Material 信息不全。但重启调试之后又可以了。不太理解

## 添加 GBuffer

### 声明并初始化 GBufferTexture

- **SceneTexturesConfig.h** 添加对应的 texture 声明

```C++
/** A uniform buffer containing common scene textures used by materials or global shaders. */
BEGIN_GLOBAL_SHADER_PARAMETER_STRUCT(FSceneTextureUniformParameters, ENGINE_API)
	// Scene Color / Depth
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, SceneColorTexture)
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, SceneDepthTexture)

	// GBuffer
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, GBufferATexture)
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, GBufferBTexture)
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, GBufferCTexture)
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, GBufferDTexture)
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, GBufferETexture)
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, GBufferFTexture)
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, GBufferVelocityTexture)

	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, GBufferCelshadingTexture) // Celshading Lucas: Add a new celshading scene texture
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, GBufferInfraredTexture) // Celshading Lucas: Add a new celshading scene texture

	// SSAO
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, ScreenSpaceAOTexture)

	// Custom Depth / Stencil
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, CustomDepthTexture)
	SHADER_PARAMETER_RDG_TEXTURE_SRV(Texture2D<uint2>, CustomStencilTexture)

	// Misc
	SHADER_PARAMETER_SAMPLER(SamplerState, PointClampSampler)
END_GLOBAL_SHADER_PARAMETER_STRUCT()
```

- **SceneTexturesConfig.cpp** 初始化绑定 GBuffer

```C++
void FSceneTexturesConfig::Init(const FSceneTexturesConfigInitSettings& InitSettings)
{
	FeatureLevel			= InitSettings.FeatureLevel;
	ShadingPath				= FSceneInterface::GetShadingPath(FeatureLevel);
	ShaderPlatform			= GetFeatureLevelShaderPlatform(FeatureLevel);
	Extent					= InitSettings.Extent;
	NumSamples				= GetDefaultMSAACount(FeatureLevel, GDynamicRHI->RHIGetPlatformTextureMaxSampleCount());
	EditorPrimitiveNumSamples = GetEditorPrimitiveNumSamples(FeatureLevel);
	ColorFormat				= PF_Unknown;
	ColorClearValue			= FClearValueBinding::Black;
	DepthClearValue			= FClearValueBinding::DepthFar;
	bRequiresAlphaChannel	= InitSettings.bRequiresAlphaChannel;
	bRequireMultiView		= InitSettings.bRequireMultiView;
	bIsUsingGBuffers		= IsUsingGBuffers(ShaderPlatform);
	bSupportsXRTargetManagerDepthAlloc = InitSettings.bSupportsXRTargetManagerDepthAlloc;
    ExtraSceneColorCreateFlags = InitSettings.ExtraSceneColorCreateFlags;
    ExtraSceneDepthCreateFlags = InitSettings.ExtraSceneDepthCreateFlags;
    
    BuildSceneColorAndDepthFlags();

	if (bIsUsingGBuffers)
	{
		FGBufferParams DefaultParams = FShaderCompileUtilities::FetchGBufferParamsRuntime(ShaderPlatform, GBL_Default);

		// GBuffer configuration information is expensive to compute, the results are cached between runs.
		struct FGBufferBindingCache
		{
			FGBufferParams GBufferParams;
			FGBufferBindings Bindings[GBL_Num];
			bool bInitialized = false;
		};
		static FGBufferBindingCache BindingCache;

		if (!BindingCache.bInitialized || BindingCache.GBufferParams != DefaultParams)
		{
			for (uint32 Layout = 0; Layout < GBL_Num; ++Layout)
			{
				GBufferParams[Layout] = (Layout == GBL_Default) ? DefaultParams : FShaderCompileUtilities::FetchGBufferParamsRuntime(ShaderPlatform, (EGBufferLayout)Layout);
				const FGBufferInfo GBufferInfo = FetchFullGBufferInfo(GBufferParams[Layout]);

				BindingCache.Bindings[Layout].GBufferA = FindGBufferBindingByName(GBufferInfo, TEXT("GBufferA"));
				BindingCache.Bindings[Layout].GBufferB = FindGBufferBindingByName(GBufferInfo, TEXT("GBufferB"));
				BindingCache.Bindings[Layout].GBufferC = FindGBufferBindingByName(GBufferInfo, TEXT("GBufferC"));
				BindingCache.Bindings[Layout].GBufferD = FindGBufferBindingByName(GBufferInfo, TEXT("GBufferD"));
				BindingCache.Bindings[Layout].GBufferE = FindGBufferBindingByName(GBufferInfo, TEXT("GBufferE"));
				BindingCache.Bindings[Layout].GBufferVelocity = FindGBufferBindingByName(GBufferInfo, TEXT("Velocity"));

				BindingCache.Bindings[Layout].GBufferCelshading = FindGBufferBindingByName(GBufferInfo, TEXT("GBufferCelshading")); // Celshading Lucas: Bind GBufferCelshading to scene texture
				BindingCache.Bindings[Layout].GBufferInfrared = FindGBufferBindingByName(GBufferInfo, TEXT("GBufferInfrared")); // Celshading Lucas: Bind GBufferCelshading to scene texture

			}

			BindingCache.GBufferParams = DefaultParams;
			BindingCache.bInitialized = true;
		}

		for (uint32 Layout = 0; Layout < GBL_Num; ++Layout)
		{
			GBufferBindings[Layout] = BindingCache.Bindings[Layout];
		}
	}
}

uint32 FSceneTexturesConfig::GetGBufferRenderTargetsInfo(FGraphicsPipelineRenderTargetsInfo& RenderTargetsInfo, EGBufferLayout Layout) const 
{
	// Assume 1 sample for now
	RenderTargetsInfo.NumSamples = 1;

	uint32 RenderTargetCount = 0;

	// All configurations use scene color in the first slot.
	RenderTargetsInfo.RenderTargetFormats[RenderTargetCount] = ColorFormat;
	RenderTargetsInfo.RenderTargetFlags[RenderTargetCount++] = ColorCreateFlags;

	// Setup the other render targets
	if (bIsUsingGBuffers)
	{
		const auto IncludeBindingIfValid = [&RenderTargetsInfo, &RenderTargetCount](const FGBufferBinding& GBufferBinding)
		{
			if (GBufferBinding.Index > 0)
			{
				RenderTargetsInfo.RenderTargetFormats[GBufferBinding.Index] = GBufferBinding.Format;
				RenderTargetsInfo.RenderTargetFlags[GBufferBinding.Index] = GBufferBinding.Flags;
				RenderTargetCount = FMath::Max(RenderTargetCount, (uint32)GBufferBinding.Index + 1);
			}
		};

		const FGBufferBindings& Bindings = GBufferBindings[Layout];
		IncludeBindingIfValid(Bindings.GBufferA);
		IncludeBindingIfValid(Bindings.GBufferB);
		IncludeBindingIfValid(Bindings.GBufferC);
		IncludeBindingIfValid(Bindings.GBufferD);
		IncludeBindingIfValid(Bindings.GBufferE);
		IncludeBindingIfValid(Bindings.GBufferVelocity);

		IncludeBindingIfValid(Bindings.GBufferCelshading); // Celshading Lucas: include GBufferCelshading binding to the render targets info result
		IncludeBindingIfValid(Bindings.GBufferInfrared); // Celshading Lucas: include GBufferCelshading binding to the render targets info result
	}
	// Forward shading path. Simple forward shading does not use velocity.
	else if (IsUsingBasePassVelocity(ShaderPlatform))
	{
		const FGBufferBinding& GBufferVelocity = GBufferBindings[GBL_Default].GBufferVelocity;
		RenderTargetsInfo.RenderTargetFormats[RenderTargetCount] = GBufferVelocity.Format;
		RenderTargetsInfo.RenderTargetFlags[RenderTargetCount++] = GBufferVelocity.Flags;
	}

	// Store final number of render targets
	RenderTargetsInfo.RenderTargetsEnabled = RenderTargetCount;

	// Precache TODO: other flags
	RenderTargetsInfo.MultiViewCount = 0;// RenderTargets.MultiViewCount;
	RenderTargetsInfo.bHasFragmentDensityAttachment = false; // RenderTargets.ShadingRateTexture != nullptr;

	return RenderTargetCount;
}
```

- **SceneTextures.cpp** 初始化 texture

```C++
void SetupSceneTextureUniformParameters(
	FRDGBuilder& GraphBuilder,
	const FSceneTextures* SceneTextures,
	ERHIFeatureLevel::Type FeatureLevel,
	ESceneTextureSetupMode SetupMode,
	FSceneTextureUniformParameters& SceneTextureParameters)
{
	const FRDGSystemTextures& SystemTextures = FRDGSystemTextures::Get(GraphBuilder);

	SceneTextureParameters.PointClampSampler = TStaticSamplerState<SF_Point>::GetRHI();
	SceneTextureParameters.SceneColorTexture = SystemTextures.Black;
	SceneTextureParameters.SceneDepthTexture = SystemTextures.DepthDummy;
	SceneTextureParameters.GBufferATexture = SystemTextures.Black;
	SceneTextureParameters.GBufferBTexture = SystemTextures.Black;
	SceneTextureParameters.GBufferCTexture = SystemTextures.Black;
	SceneTextureParameters.GBufferDTexture = SystemTextures.Black;
	SceneTextureParameters.GBufferETexture = SystemTextures.Black;
	SceneTextureParameters.GBufferFTexture = SystemTextures.MidGrey;
	SceneTextureParameters.GBufferVelocityTexture = SystemTextures.Black;
	SceneTextureParameters.ScreenSpaceAOTexture = GetScreenSpaceAOFallback(SystemTextures);
	SceneTextureParameters.CustomDepthTexture = SystemTextures.DepthDummy;
	SceneTextureParameters.CustomStencilTexture = SystemTextures.StencilDummySRV;

	SceneTextureParameters.GBufferCelshadingTexture = SystemTextures.Black; // initialize celshading gbuffer texture
	SceneTextureParameters.GBufferInfraredTexture = SystemTextures.Black; // initialize celshading gbuffer texture

	if (SceneTextures)
	{
		const EShaderPlatform ShaderPlatform = SceneTextures->Config.ShaderPlatform;

		if (EnumHasAnyFlags(SetupMode, ESceneTextureSetupMode::SceneColor))
		{
			SceneTextureParameters.SceneColorTexture = SceneTextures->Color.Resolve;
		}

		if (EnumHasAnyFlags(SetupMode, ESceneTextureSetupMode::SceneDepth))
		{
			SceneTextureParameters.SceneDepthTexture = SceneTextures->Depth.Resolve;
		}

		if (IsUsingGBuffers(ShaderPlatform))
		{
			if (EnumHasAnyFlags(SetupMode, ESceneTextureSetupMode::GBufferA) && HasBeenProduced(SceneTextures->GBufferA))
			{
				SceneTextureParameters.GBufferATexture = SceneTextures->GBufferA;
			}

			if (EnumHasAnyFlags(SetupMode, ESceneTextureSetupMode::GBufferB) && HasBeenProduced(SceneTextures->GBufferB))
			{
				SceneTextureParameters.GBufferBTexture = SceneTextures->GBufferB;
			}

			if (EnumHasAnyFlags(SetupMode, ESceneTextureSetupMode::GBufferC) && HasBeenProduced(SceneTextures->GBufferC))
			{
				SceneTextureParameters.GBufferCTexture = SceneTextures->GBufferC;
			}

			if (EnumHasAnyFlags(SetupMode, ESceneTextureSetupMode::GBufferD) && HasBeenProduced(SceneTextures->GBufferD))
			{
				SceneTextureParameters.GBufferDTexture = SceneTextures->GBufferD;
			}

			if (EnumHasAnyFlags(SetupMode, ESceneTextureSetupMode::GBufferE) && HasBeenProduced(SceneTextures->GBufferE))
			{
				SceneTextureParameters.GBufferETexture = SceneTextures->GBufferE;
			}

			if (EnumHasAnyFlags(SetupMode, ESceneTextureSetupMode::GBufferF) && HasBeenProduced(SceneTextures->GBufferF))
			{
				SceneTextureParameters.GBufferFTexture = SceneTextures->GBufferF;
			}

			// Celshading Lucas : if produced, bind the celshading RDG texture to the shader texture
			if (EnumHasAnyFlags(SetupMode, ESceneTextureSetupMode::GBufferCelshading) && HasBeenProduced(SceneTextures->GBufferCelshading))
			{
				SceneTextureParameters.GBufferCelshadingTexture = SceneTextures->GBufferCelshading;
			}

			if (EnumHasAnyFlags(SetupMode, ESceneTextureSetupMode::GBufferInfrared) && HasBeenProduced(SceneTextures->GBufferInfrared))
			{
				SceneTextureParameters.GBufferInfraredTexture = SceneTextures->GBufferInfrared;
			}
		}
	}
}
```

- **SceneTextureParameters.h** 添加 pass parameter struct

```C++
BEGIN_SHADER_PARAMETER_STRUCT(FSceneTextureParameters, )
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, SceneDepthTexture)
	SHADER_PARAMETER_RDG_TEXTURE_SRV(Texture2D<uint2>, SceneStencilTexture)
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, GBufferATexture)
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, GBufferBTexture)
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, GBufferCTexture)
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, GBufferDTexture)
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, GBufferETexture)
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, GBufferFTexture)
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, GBufferVelocityTexture)

	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, GBufferCelshadingTexture) // Celshading Lucas: Add GBufferCelshadingTexture to render target parameters
	SHADER_PARAMETER_RDG_TEXTURE(Texture2D, GBufferInfraredTexture) // Celshading Lucas: Add GBufferCelshadingTexture to render target parameters

END_SHADER_PARAMETER_STRUCT()
````

- **PlanarReflectionRendering.cpp** 在要用到的地方添加上这个 parameter 的引用

```C++
void FDeferredShadingSceneRenderer::RenderDeferredPlanarReflections(FRDGBuilder& GraphBuilder, const FSceneTextureParameters& SceneTextures, const FViewInfo& View, FRDGTextureRef& ReflectionsOutputTexture)
{
	check(HasDeferredPlanarReflections(View));

	// Allocate planar reflection texture
	bool bClearReflectionsOutputTexture = false;
	if (!ReflectionsOutputTexture)
	{
		FRDGTextureDesc Desc = FRDGTextureDesc::Create2D(
			SceneTextures.SceneDepthTexture->Desc.Extent,
			PF_FloatRGBA, FClearValueBinding(FLinearColor(0, 0, 0, 0)),
			TexCreate_ShaderResource | TexCreate_RenderTargetable);

		ReflectionsOutputTexture = GraphBuilder.CreateTexture(Desc, TEXT("PlanarReflections"));
		bClearReflectionsOutputTexture = true;
	}

	FPlanarReflectionPS::FParameters* PassParameters = GraphBuilder.AllocParameters<FPlanarReflectionPS::FParameters>();
	PassParameters->SceneTextures.SceneDepthTexture = SceneTextures.SceneDepthTexture;
	PassParameters->SceneTextures.GBufferATexture = SceneTextures.GBufferATexture;
	PassParameters->SceneTextures.GBufferBTexture = SceneTextures.GBufferBTexture;

	PassParameters->SceneTextures.GBufferCTexture = SceneTextures.GBufferCTexture;
	PassParameters->SceneTextures.GBufferDTexture = SceneTextures.GBufferDTexture;
	PassParameters->SceneTextures.GBufferETexture = SceneTextures.GBufferETexture;
	PassParameters->SceneTextures.GBufferFTexture = SceneTextures.GBufferFTexture;
	PassParameters->SceneTextures.GBufferVelocityTexture = SceneTextures.GBufferVelocityTexture;

	PassParameters->SceneTextures.GBufferCelshadingTexture = SceneTextures.GBufferCelshadingTexture; // Celshading Lucas: pass scene texture parameters to planar reflection renderer
	PassParameters->SceneTextures.GBufferInfraredTexture = SceneTextures.GBufferInfraredTexture; // Celshading Lucas: pass scene texture parameters to planar reflection renderer
}
````

- **PostProcessBufferInspector.cpp**

```C++
FScreenPassTexture AddPixelInspectorPass(FRDGBuilder& GraphBuilder, const FViewInfo& View, const FPixelInspectorInputs& Inputs)
{
	check(Inputs.SceneColor.IsValid());
	check(Inputs.SceneColor.ViewRect == Inputs.SceneColorBeforeTonemap.ViewRect);
	check(Inputs.OriginalSceneColor.IsValid());
	check(Inputs.OriginalSceneColor.ViewRect == View.ViewRect);
	check(View.bUsePixelInspector);

	RDG_EVENT_SCOPE(GraphBuilder, "PixelInspector");

	// Perform copies of scene textures data into staging resources for visualization.
	{
		FSceneTextureParameters SceneTextures = GetSceneTextureParameters(GraphBuilder, View);

		// GBufferF is optional, so it may be a dummy texture. Revert it to null if so.
		if (SceneTextures.GBufferFTexture->Desc.Extent != Inputs.OriginalSceneColor.Texture->Desc.Extent)
		{
			SceneTextures.GBufferFTexture = nullptr;
		}

		FPixelInspectorParameters* PassParameters = GraphBuilder.AllocParameters<FPixelInspectorParameters>();
		PassParameters->GBufferA = SceneTextures.GBufferATexture;
		PassParameters->GBufferB = SceneTextures.GBufferBTexture;
		PassParameters->GBufferC = SceneTextures.GBufferCTexture;
		PassParameters->GBufferD = SceneTextures.GBufferDTexture;
		PassParameters->GBufferE = SceneTextures.GBufferETexture;
		PassParameters->GBufferF = SceneTextures.GBufferFTexture;
		PassParameters->SceneColor = Inputs.SceneColor.Texture;
		PassParameters->SceneColorBeforeTonemap = Inputs.SceneColorBeforeTonemap.Texture;
		PassParameters->SceneDepth = SceneTextures.SceneDepthTexture;
		PassParameters->OriginalSceneColor = Inputs.OriginalSceneColor.Texture;
		
		PassParameters->GBufferCelshading = SceneTextures.GBufferCelshadingTexture; // Celshading Lucas: pass scene texture parameter to pixel inspector
		PassParameters->GBufferInfrared = SceneTextures.GBufferInfraredTexture; // Celshading Lucas: pass scene texture parameter to pixel inspector
	}
}
```

- **SceneTextureParameters.cpp**

```C++
FSceneTextureParameters GetSceneTextureParameters(FRDGBuilder& GraphBuilder, const FSceneTextures& SceneTextures)
{
	const auto& SystemTextures = FRDGSystemTextures::Get(GraphBuilder);

	FSceneTextureParameters Parameters;

	// Should always have a depth buffer around allocated, since early z-pass is first.
	Parameters.SceneDepthTexture = SceneTextures.Depth.Resolve;
	Parameters.SceneStencilTexture = GraphBuilder.CreateSRV(FRDGTextureSRVDesc::CreateWithPixelFormat(Parameters.SceneDepthTexture, PF_X24_G8));

	// Registers all the scene texture from the scene context. No fallback is provided to catch mistake at shader parameter validation time
	// when a pass is trying to access a resource before any other pass actually created it.
	Parameters.GBufferVelocityTexture = GetIfProduced(SceneTextures.Velocity);
	Parameters.GBufferATexture = GetIfProduced(SceneTextures.GBufferA);
	Parameters.GBufferBTexture = GetIfProduced(SceneTextures.GBufferB);
	Parameters.GBufferCTexture = GetIfProduced(SceneTextures.GBufferC);
	Parameters.GBufferDTexture = GetIfProduced(SceneTextures.GBufferD);
	Parameters.GBufferETexture = GetIfProduced(SceneTextures.GBufferE);
	Parameters.GBufferFTexture = GetIfProduced(SceneTextures.GBufferF, SystemTextures.MidGrey);

	Parameters.GBufferCelshadingTexture = GetIfProduced(SceneTextures.GBufferCelshading); // Celshading Lucas: Get Celshading scene texture
	Parameters.GBufferInfraredTexture = GetIfProduced(SceneTextures.GBufferInfrared); // Celshading Lucas: Get Celshading scene texture

	return Parameters;
}

FSceneTextureParameters GetSceneTextureParameters(FRDGBuilder& GraphBuilder, TRDGUniformBufferRef<FSceneTextureUniformParameters> SceneTextureUniformBuffer)
{
	FSceneTextureParameters Parameters;
	Parameters.SceneDepthTexture = (*SceneTextureUniformBuffer)->SceneDepthTexture;
	if (Parameters.SceneDepthTexture)
	{
		Parameters.SceneStencilTexture = GraphBuilder.CreateSRV(FRDGTextureSRVDesc::CreateWithPixelFormat(Parameters.SceneDepthTexture, PF_X24_G8));
	}
	Parameters.GBufferATexture = (*SceneTextureUniformBuffer)->GBufferATexture;
	Parameters.GBufferBTexture = (*SceneTextureUniformBuffer)->GBufferBTexture;
	Parameters.GBufferCTexture = (*SceneTextureUniformBuffer)->GBufferCTexture;
	Parameters.GBufferDTexture = (*SceneTextureUniformBuffer)->GBufferDTexture;
	Parameters.GBufferETexture = (*SceneTextureUniformBuffer)->GBufferETexture;
	Parameters.GBufferFTexture = (*SceneTextureUniformBuffer)->GBufferFTexture;
	Parameters.GBufferVelocityTexture = (*SceneTextureUniformBuffer)->GBufferVelocityTexture;
	
	Parameters.GBufferCelshadingTexture = (*SceneTextureUniformBuffer)->GBufferCelshadingTexture; // Celshading Lucas: Get Celshading scene texture
	Parameters.GBufferInfraredTexture = (*SceneTextureUniformBuffer)->GBufferInfraredTexture; // Celshading Lucas: Get Celshading scene texture

	return Parameters;
}
```

### 创建和初始化 GBuffer

- 创建对应的 RDG texture，**SceneTextures.h**

```C++
struct RENDERER_API FSceneTextures : public FMinimalSceneTextures
{
	// Initializes the scene textures structure in the FViewFamilyInfo
	static void InitializeViewFamily(FRDGBuilder& GraphBuilder, FViewFamilyInfo& ViewFamily);

	// Configures an array of render targets for the GBuffer pass.
	uint32 GetGBufferRenderTargets(
		TStaticArray<FTextureRenderTargetBinding,
		MaxSimultaneousRenderTargets>& RenderTargets,
		EGBufferLayout Layout = GBL_Default) const;
	uint32 GetGBufferRenderTargets(
		ERenderTargetLoadAction LoadAction,
		FRenderTargetBindingSlots& RenderTargets,
		EGBufferLayout Layout = GBL_Default) const;

	// (Deferred) Texture containing conservative downsampled depth for occlusion.
	FRDGTextureRef SmallDepth{};

	// (Deferred) Textures containing geometry information for deferred shading.
	FRDGTextureRef GBufferA{};
	FRDGTextureRef GBufferB{};
	FRDGTextureRef GBufferC{};
	FRDGTextureRef GBufferD{};
	FRDGTextureRef GBufferE{};
	FRDGTextureRef GBufferF{};

	FRDGTextureRef GBufferCelshading{}; // Celshading Lucas: create the RDG Texture
	FRDGTextureRef GBufferInfrared{}; // Celshading Lucas: create the RDG Texture
}
```

- **SceneRenderTargetParameters.h** 添加 setupmode

```C++
enum class ESceneTextureSetupMode : uint32
{
	None			= 0,
	SceneColor		= 1 << 0,
	SceneDepth		= 1 << 1,
	SceneVelocity	= 1 << 2,
	GBufferA		= 1 << 3,
	GBufferB		= 1 << 4,
	GBufferC		= 1 << 5,
	GBufferD		= 1 << 6,
	GBufferE		= 1 << 7,
	GBufferF		= 1 << 8,

	GBufferCelshading = 1 << 9, // Celshading Lucas: Append celshading gbuffer to scene texture setup mode
	GBufferInfrared = 1 << 10,

	SSAO			= 1 << 11, // 9
	CustomDepth		= 1 << 12, // 10
	GBuffers		= GBufferA | GBufferB | GBufferC | GBufferD | GBufferE | GBufferF | GBufferCelshading | GBufferInfrared, // | GBufferCelshading
	All				= SceneColor | SceneDepth | SceneVelocity | GBuffers | SSAO | CustomDepth
};
```

- ** SceneRendering.h** 添加 flag 设置 VRAM

```C++
struct FFastVramConfig
{
	FFastVramConfig();
	void Update();
	void OnCVarUpdated();
	void OnSceneRenderTargetsAllocated();

	ETextureCreateFlags GBufferA;
	ETextureCreateFlags GBufferB;
	ETextureCreateFlags GBufferC;
	ETextureCreateFlags GBufferD;
	ETextureCreateFlags GBufferE;
	ETextureCreateFlags GBufferF;
	ETextureCreateFlags GBufferVelocity;

	ETextureCreateFlags GBufferCelshading; // Celshading Lucas: enable a flag for using fast vram for this gbuffer texture
	ETextureCreateFlags GBufferInfrared; // Celshading Lucas: enable a flag for using fast vram for this gbuffer texture
}
```

- **SceneRendering.cpp** 添加 CVAR

```C++
FASTVRAM_CVAR(GBufferInfrared, 0); // Celshading Lucas: Create fast VRAM CVAR and assign it to 0 (false)
```

- **TextureShareSceneViewExtension.cpp TextureShareStrings.h** 声明避免错误

```C++
void FTextureShareSceneViewExtension::ShareSceneViewColors_RenderThread(FRDGBuilder& GraphBuilder, const FSceneTextures& SceneTextures, const FTextureShareSceneView& InView)
{
	const auto AddShareTexturePass = [&](const TCHAR* InTextureName, const FRDGTextureRef& InTextureRef)
	{
		// Send resource
		ObjectProxy->ShareResource_RenderThread(GraphBuilder, FTextureShareCoreResourceDesc(InTextureName, InView.ViewInfo.ViewDesc, ETextureShareTextureOp::Read),
			InTextureRef, InView.GPUIndex, &InView.UnconstrainedViewRect);
	};

	AddShareTexturePass(TextureShareStrings::SceneTextures::SceneColor, SceneTextures.Color.Resolve);

	AddShareTexturePass(TextureShareStrings::SceneTextures::SceneDepth, SceneTextures.Depth.Resolve);
	AddShareTexturePass(TextureShareStrings::SceneTextures::SmallDepthZ, SceneTextures.SmallDepth);

	AddShareTexturePass(TextureShareStrings::SceneTextures::GBufferA, SceneTextures.GBufferA);
	AddShareTexturePass(TextureShareStrings::SceneTextures::GBufferB, SceneTextures.GBufferB);
	AddShareTexturePass(TextureShareStrings::SceneTextures::GBufferC, SceneTextures.GBufferC);
	AddShareTexturePass(TextureShareStrings::SceneTextures::GBufferD, SceneTextures.GBufferD);
	AddShareTexturePass(TextureShareStrings::SceneTextures::GBufferE, SceneTextures.GBufferE);
	AddShareTexturePass(TextureShareStrings::SceneTextures::GBufferF, SceneTextures.GBufferF);

	AddShareTexturePass(TextureShareStrings::SceneTextures::GBufferCelshading, SceneTextures.GBufferCelshading);
	AddShareTexturePass(TextureShareStrings::SceneTextures::GBufferInfrared, SceneTextures.GBufferInfrared);

}

namespace TextureShareStrings
{
	namespace SceneTextures
	{
		// Read-only scene resources
		static constexpr auto SceneColor = TEXT("SceneColor");
		static constexpr auto SceneDepth = TEXT("SceneDepth");
		static constexpr auto SmallDepthZ = TEXT("SmallDepthZ");
		static constexpr auto GBufferA = TEXT("GBufferA");
		static constexpr auto GBufferB = TEXT("GBufferB");
		static constexpr auto GBufferC = TEXT("GBufferC");
		static constexpr auto GBufferD = TEXT("GBufferD");
		static constexpr auto GBufferE = TEXT("GBufferE");
		static constexpr auto GBufferF = TEXT("GBufferF");

		static constexpr auto GBufferCelshading = TEXT("GBufferCelshading");
		static constexpr auto GBufferInfrared = TEXT("GBufferInfrared");

		// Read-write RTTs
		static constexpr auto FinalColor = TEXT("FinalColor");
		static constexpr auto Backbuffer = TEXT("Backbuffer");
	}
};
```

- ** PostProcessBufferInspector.cpp** 添加对应的 parameter

````C++
BEGIN_SHADER_PARAMETER_STRUCT(FPixelInspectorParameters, )
	RDG_TEXTURE_ACCESS(GBufferA, ERHIAccess::CopySrc)
	RDG_TEXTURE_ACCESS(GBufferB, ERHIAccess::CopySrc)
	RDG_TEXTURE_ACCESS(GBufferC, ERHIAccess::CopySrc)
	RDG_TEXTURE_ACCESS(GBufferD, ERHIAccess::CopySrc)
	RDG_TEXTURE_ACCESS(GBufferE, ERHIAccess::CopySrc)
	RDG_TEXTURE_ACCESS(GBufferF, ERHIAccess::CopySrc)
	RDG_TEXTURE_ACCESS(SceneColor, ERHIAccess::CopySrc)
	RDG_TEXTURE_ACCESS(SceneColorBeforeTonemap, ERHIAccess::CopySrc)
	RDG_TEXTURE_ACCESS(SceneDepth, ERHIAccess::CopySrc)
	RDG_TEXTURE_ACCESS(OriginalSceneColor, ERHIAccess::CopySrc)

	RDG_TEXTURE_ACCESS(GBufferCelshading, ERHIAccess::CopySrc) // Celshading Lucas: add a copy of gbuffer celshading to pixel inspector (?)
	RDG_TEXTURE_ACCESS(GBufferInfrared, ERHIAccess::CopySrc) // Celshading Lucas: add a copy of gbuffer celshading to pixel inspector (?)

END_SHADER_PARAMETER_STRUCT()

void ProcessPixelInspectorRequests(
	FRHICommandListImmediate& RHICmdList,
	const FViewInfo& View,
	const FPixelInspectorParameters& Parameters,
	const FIntRect SceneColorViewRect)
{
	if (Parameters.GBufferInfrared)
	{
		FRHITexture* SourceBufferInfrared = Parameters.GBufferInfrared->GetRHI();
		if (DestinationBufferBCDEF->GetFormat() == SourceBufferInfrared->GetFormat())
		{
			FRHICopyTextureInfo CopyInfo;
			CopyInfo.SourcePosition = SourcePoint;
			CopyInfo.DestPosition = FIntVector(5, 0, 0);
			CopyInfo.Size = FIntVector(1, 1, 1);
			RHICmdList.CopyTexture(SourceBufferInfrared, DestinationBufferBCDEF, CopyInfo);
		}
	}
}
```

### 生成 GBuffer

- **GBufferInfo.h** 添加绑定信息

```C++
struct FGBufferBindings
{
	FGBufferBinding GBufferA;
	FGBufferBinding GBufferB;
	FGBufferBinding GBufferC;
	FGBufferBinding GBufferD;
	FGBufferBinding GBufferE;
	FGBufferBinding GBufferVelocity;

	FGBufferBinding GBufferCelshading;
	FGBufferBinding GBufferInfrared;
};
```

- **SceneTextures.cpp** 添加绑定和初始化

```C++
int32 FSceneTextures::GetGBufferRenderTargets(
	TStaticArray<FTextureRenderTargetBinding,
	MaxSimultaneousRenderTargets>& RenderTargets,
	EGBufferLayout Layout) const
{
	uint32 RenderTargetCount = 0;

	// All configurations use scene color in the first slot.
	RenderTargets[RenderTargetCount++] = FTextureRenderTargetBinding(Color.Target);

	if (Config.bIsUsingGBuffers)
	{
		struct FGBufferEntry
		{
			FGBufferEntry(const TCHAR* InName, FRDGTextureRef InTexture, int32 InIndex)
				: Name(InName)
				, Texture(InTexture)
				, Index(InIndex)
			{}

			const TCHAR* Name;
			FRDGTextureRef Texture;
			int32 Index;
		};

		const FGBufferBindings& Bindings = Config.GBufferBindings[Layout];
		const FGBufferEntry GBufferEntries[] =
		{
			{ TEXT("GBufferA"), GBufferA, Bindings.GBufferA.Index },
			{ TEXT("GBufferB"), GBufferB, Bindings.GBufferB.Index },
			{ TEXT("GBufferC"), GBufferC, Bindings.GBufferC.Index },
			{ TEXT("GBufferD"), GBufferD, Bindings.GBufferD.Index },
			{ TEXT("GBufferE"), GBufferE, Bindings.GBufferE.Index },
			{ TEXT("Velocity"), Velocity, Bindings.GBufferVelocity.Index },

			{ TEXT("GBufferCelshading"), GBufferCelshading, Bindings.GBufferCelshading.Index },
			{ TEXT("GBufferInfrared"), GBufferInfrared, Bindings.GBufferInfrared.Index }
		};
	}
}

void FSceneTextures::InitializeViewFamily(FRDGBuilder& GraphBuilder, FViewFamilyInfo& ViewFamily)
{
	const FSceneTexturesConfig& Config = ViewFamily.SceneTexturesConfig;
	FSceneTextures& SceneTextures = ViewFamily.SceneTextures;

	FMinimalSceneTextures::InitializeViewFamily(GraphBuilder, ViewFamily);

	if (Config.ShadingPath == EShadingPath::Deferred)
	{
		// Screen Space Ambient Occlusion
		SceneTextures.ScreenSpaceAO = CreateScreenSpaceAOTexture(GraphBuilder, Config.Extent);

		// Small Depth
		const FIntPoint SmallDepthExtent = GetDownscaledExtent(Config.Extent, Config.SmallDepthDownsampleFactor);
		const FRDGTextureDesc SmallDepthDesc(FRDGTextureDesc::Create2D(SmallDepthExtent, PF_DepthStencil, FClearValueBinding::None, TexCreate_DepthStencilTargetable | TexCreate_ShaderResource));
		SceneTextures.SmallDepth = GraphBuilder.CreateTexture(SmallDepthDesc, TEXT("SmallDepthZ"));
	}
	else
	{
		// Mobile Screen Space Ambient Occlusion
		SceneTextures.ScreenSpaceAO = CreateMobileScreenSpaceAOTexture(GraphBuilder, Config);

		if (Config.MobilePixelProjectedReflectionExtent != FIntPoint::ZeroValue)
		{
			SceneTextures.PixelProjectedReflection = CreateMobilePixelProjectedReflectionTexture(GraphBuilder, Config.MobilePixelProjectedReflectionExtent);
		}
	}

	// Velocity
	SceneTextures.Velocity = GraphBuilder.CreateTexture(FVelocityRendering::GetRenderTargetDesc(Config.ShaderPlatform, Config.Extent), TEXT("SceneVelocity"));

	if (Config.bIsUsingGBuffers)
	{
		ETextureCreateFlags FlagsToAdd = TexCreate_None;
		const FGBufferBindings& Bindings = Config.GBufferBindings[GBL_Default];

		if (Bindings.GBufferA.Index >= 0)
		{
			const FRDGTextureDesc Desc(FRDGTextureDesc::Create2D(Config.Extent, Bindings.GBufferA.Format, FClearValueBinding::Transparent, Bindings.GBufferA.Flags | FlagsToAdd | GFastVRamConfig.GBufferA));
			SceneTextures.GBufferA = GraphBuilder.CreateTexture(Desc, TEXT("GBufferA"));
		}

		if (Bindings.GBufferB.Index >= 0)
		{
			const FRDGTextureDesc Desc(FRDGTextureDesc::Create2D(Config.Extent, Bindings.GBufferB.Format, FClearValueBinding::Transparent, Bindings.GBufferB.Flags | FlagsToAdd | GFastVRamConfig.GBufferB));
			SceneTextures.GBufferB = GraphBuilder.CreateTexture(Desc, TEXT("GBufferB"));
		}

		if (Bindings.GBufferC.Index >= 0)
		{
			const FRDGTextureDesc Desc(FRDGTextureDesc::Create2D(Config.Extent, Bindings.GBufferC.Format, FClearValueBinding::Transparent, Bindings.GBufferC.Flags | FlagsToAdd | GFastVRamConfig.GBufferC));
			SceneTextures.GBufferC = GraphBuilder.CreateTexture(Desc, TEXT("GBufferC"));
		}

		if (Bindings.GBufferD.Index >= 0)
		{
			const FRDGTextureDesc Desc(FRDGTextureDesc::Create2D(Config.Extent, Bindings.GBufferD.Format, FClearValueBinding::Transparent, Bindings.GBufferD.Flags | FlagsToAdd | GFastVRamConfig.GBufferD));
			SceneTextures.GBufferD = GraphBuilder.CreateTexture(Desc, TEXT("GBufferD"));
		}

		if (Bindings.GBufferE.Index >= 0)
		{
			const FRDGTextureDesc Desc(FRDGTextureDesc::Create2D(Config.Extent, Bindings.GBufferE.Format, FClearValueBinding::Transparent, Bindings.GBufferE.Flags | FlagsToAdd | GFastVRamConfig.GBufferE));
			SceneTextures.GBufferE = GraphBuilder.CreateTexture(Desc, TEXT("GBufferE"));
		}

		// Celshading Lucas: create our celshading Gbuffer RDG texture with all parameters set before.
		if (Bindings.GBufferCelshading.Index >= 0)
		{
			const FRDGTextureDesc Desc(FRDGTextureDesc::Create2D(Config.Extent, Bindings.GBufferCelshading.Format, FClearValueBinding::Transparent, Bindings.GBufferCelshading.Flags | FlagsToAdd | GFastVRamConfig.GBufferCelshading));
			SceneTextures.GBufferCelshading = GraphBuilder.CreateTexture(Desc, TEXT("GBufferCelshading"));
		}

		if (Bindings.GBufferInfrared.Index >= 0)
		{
			const FRDGTextureDesc Desc(FRDGTextureDesc::Create2D(Config.Extent, Bindings.GBufferInfrared.Format, FClearValueBinding::Transparent, Bindings.GBufferInfrared.Flags | FlagsToAdd | GFastVRamConfig.GBufferInfrared));
			SceneTextures.GBufferInfrared = GraphBuilder.CreateTexture(Desc, TEXT("GBufferInfrared"));
		}
	}
}
```

- **GBufferInfo.h GBufferInfo.cpp** 添加 slot

```C++
enum EGBufferSlot
{
	GBS_Invalid,
	GBS_SceneColor, // RGB 11.11.10
	GBS_WorldNormal, // RGB 10.10.10
	GBS_PerObjectGBufferData, // 2
	GBS_Metallic, // R8
	GBS_Specular, // R8
	GBS_Roughness, // R8
	GBS_ShadingModelId, // 4 bits
	GBS_SelectiveOutputMask, // 4 bits
	GBS_BaseColor, // RGB8
	GBS_GenericAO, // R8
	GBS_IndirectIrradiance, // R8
	GBS_AO, // R8
	GBS_Velocity, // RG, float16
	GBS_PrecomputedShadowFactor, // RGBA8
	GBS_WorldTangent, // RGB8
	GBS_Anisotropy, // R8
	GBS_CustomData, // RGBA8, no compression
	GBS_SubsurfaceColor, // RGB8
	GBS_Opacity, // R8
	GBS_SubsurfaceProfile, //RGB8
	GBS_ClearCoat, // R8
	GBS_ClearCoatRoughness, // R8
	GBS_HairSecondaryWorldNormal, // RG8
	GBS_HairBacklit, // R8
	GBS_Cloth, // R8
	GBS_SubsurfaceProfileX, // R8
	GBS_IrisNormal, // RG8
	GBS_SeparatedMainDirLight, // RGB 11.11.10

	GBS_Celshading, // Celshading Lucas: R8 slot
	GBS_Infrared, // RGBA8, no compression

	GBS_Num
};

TArray < EGBufferSlot > FetchGBufferSlots(bool bHasVelocity, bool bHasTangent, bool bHasPrecShadowFactor)
{
	TArray < EGBufferSlot > NeededSlots;

	NeededSlots.Push(GBS_SceneColor);
	NeededSlots.Push(GBS_WorldNormal);
	NeededSlots.Push(GBS_PerObjectGBufferData);
	NeededSlots.Push(GBS_Metallic);
	NeededSlots.Push(GBS_Specular);
	NeededSlots.Push(GBS_Roughness);
	NeededSlots.Push(GBS_ShadingModelId);
	NeededSlots.Push(GBS_SelectiveOutputMask);
	NeededSlots.Push(GBS_BaseColor);

	// this is needed for all combinations, will have to split it later
	NeededSlots.Push(GBS_GenericAO);

	if (bHasVelocity)
	{
		NeededSlots.Push(GBS_Velocity);
	}
	if (bHasPrecShadowFactor)
	{
		NeededSlots.Push(GBS_PrecomputedShadowFactor);
	}
	if (bHasTangent)
	{
		NeededSlots.Push(GBS_WorldTangent);
		NeededSlots.Push(GBS_Anisotropy);
	}
	NeededSlots.Push(GBS_CustomData);

	NeededSlots.Push(GBS_Celshading);
	NeededSlots.Push(GBS_Infrared);

	return NeededSlots;
}
```

- **ShaderGenerationUtil.cpp** 绑定 HLSL 端的 GBuffer

```C++
static FString GetSlotTextName(EGBufferSlot Slot)
{
	FString Ret = TEXT("");
	switch (Slot)
	{
	case GBS_Invalid:
		check(0);
		return TEXT("Invalid");
	case GBS_SceneColor:
		return TEXT("SceneColor");
	case GBS_WorldNormal:
		return TEXT("WorldNormal");
	case GBS_PerObjectGBufferData:
		return TEXT("PerObjectGBufferData");
	case GBS_Metallic:
		return TEXT("Metallic");
	case GBS_Specular:
		return TEXT("Specular");
	case GBS_Roughness:
		return TEXT("Roughness");
	case GBS_ShadingModelId:
		return TEXT("ShadingModelID");
	case GBS_SelectiveOutputMask:
		return TEXT("SelectiveOutputMask");
	case GBS_BaseColor:
		return TEXT("BaseColor");
	case GBS_GenericAO:
		return TEXT("GenericAO");
	case GBS_IndirectIrradiance:
		return TEXT("IndirectIrradiance");
	case GBS_Velocity:
		return TEXT("Velocity");
	case GBS_PrecomputedShadowFactor:
		return TEXT("PrecomputedShadowFactors");
	case GBS_WorldTangent:
		return TEXT("WorldTangent");
	case GBS_Anisotropy:
		return TEXT("Anisotropy");
	case GBS_CustomData:
		return TEXT("CustomData");
	case GBS_SubsurfaceColor:
		return TEXT("SubsurfaceColor");
	case GBS_Opacity:
		return TEXT("Opacity");
	case GBS_SubsurfaceProfile:
		return TEXT("SubsurfaceProfile");
	case GBS_ClearCoat:
		return TEXT("ClearCoat");
	case GBS_ClearCoatRoughness:
		return TEXT("ClearCoatRoughness");
	case GBS_HairSecondaryWorldNormal:
		return TEXT("HaireEcondaryWorldNormal");
	case GBS_HairBacklit:
		return TEXT("HairBacklit");
	case GBS_Cloth:
		return TEXT("Cloth");
	case GBS_SubsurfaceProfileX:
		return TEXT("SubsurfaceProfileX");
	case GBS_IrisNormal:
		return TEXT("IrisNormal");
	case GBS_SeparatedMainDirLight:
		return TEXT("SeparatedMainDirLight");

	case GBS_Celshading:
		return TEXT("Celshading"); // Celshading Lucas: This is the name this slot use in the HLSL struct GBufferData
	case GBS_Infrared:
		return TEXT("Infrared"); // Celshading Lucas: This is the name this slot use in the HLSL struct GBufferData

	default:
		break;
	};

	return TEXT("ERROR");
}
```

- **GBufferInfo.cpp** 初始化 gbuffer，设置数量，添加 pack info

```C++
FGBufferInfo RENDERCORE_API FetchLegacyGBufferInfo(const FGBufferParams& Params)
{
	FGBufferInfo Info = {};

	check(!Params.bHasVelocity || !Params.bHasTangent);

	bool bStaticLighting = Params.bHasPrecShadowFactor;

	int32 TargetLighting = 0;
	int32 TargetGBufferA = 1;
	int32 TargetGBufferB = 2;
	int32 TargetGBufferC = 3;
	int32 TargetGBufferD = -1;
	int32 TargetGBufferE = -1;
	int32 TargetGBufferF = -1;
	int32 TargetVelocity = -1;

	int32 TargetCelshading = -1;
	int32 TargetInfrared = -1;
}

if (Params.bHasVelocity == 0 && Params.bHasTangent == 0)
{
	Info.NumTargets = Params.bHasPrecShadowFactor ? 8 : 7; // 6 : 5
}
else
{
	Info.NumTargets = Params.bHasPrecShadowFactor ? 9 : 8; // 7 : 6
}

// This code should match TBasePassPS
if (Params.bHasVelocity == 0 && Params.bHasTangent == 0)
{
	TargetGBufferD = 4;
	Info.Targets[4].Init(GBT_Unorm_8_8_8_8,  TEXT("GBufferD"), false,  true,  true,  true);
	
	TargetCelshading = 5;
	Info.Targets[5].Init(GBT_Unorm_8_8_8_8, TEXT("GBufferCelshading"), false, true, true, true);

	TargetInfrared = 6;
	Info.Targets[6].Init(GBT_Unorm_8_8_8_8, TEXT("GBufferInfrared"), false, true, true, true);

	TargetSeparatedMainDirLight = 7; // 5

	if (Params.bHasPrecShadowFactor)
	{
		TargetGBufferE = 7; // 5
		Info.Targets[7].Init(GBT_Unorm_8_8_8_8, TEXT("GBufferE"), false, true, true, true); // 5
		TargetSeparatedMainDirLight = 8; // 6
	}
}
else if (Params.bHasVelocity)
{
	TargetVelocity = 4;
	TargetGBufferD = 5;

	// note the false for use extra flags for velocity, not quite sure of all the ramifications, but this keeps it consistent with previous usage
	Info.Targets[4].Init(Params.bUsesVelocityDepth ? GBT_Unorm_16_16_16_16 : (IsAndroidOpenGLESPlatform(Params.ShaderPlatform) ? GBT_Float_16_16 : GBT_Unorm_16_16), TEXT("Velocity"), false, true, true, false);
	Info.Targets[5].Init(GBT_Unorm_8_8_8_8, TEXT("GBufferD"), false, true, true, true);
	
	TargetCelshading = 6;
	Info.Targets[6].Init(GBT_Unorm_8_8_8_8, TEXT("GBufferCelshading"), false, true, true, true);

	TargetInfrared = 7;
	Info.Targets[7].Init(GBT_Unorm_8_8_8_8, TEXT("GBufferInfrared"), false, true, true, true);

	TargetSeparatedMainDirLight = 8; // 6

	if (Params.bHasPrecShadowFactor)
	{
		TargetGBufferE = 8; // 6
		Info.Targets[8].Init(GBT_Unorm_8_8_8_8, TEXT("GBufferE"), false, true, true, false); // 6
		TargetSeparatedMainDirLight = 9; // 7
	}
}
else if (Params.bHasTangent)
{
	TargetGBufferF = 4;
	TargetGBufferD = 5;

	Info.Targets[4].Init(GBT_Unorm_8_8_8_8,  TEXT("GBufferF"), false,  true,  true,  true);
	Info.Targets[5].Init(GBT_Unorm_8_8_8_8, TEXT("GBufferD"), false, true, true, true);
	
	TargetCelshading = 6;
	Info.Targets[6].Init(GBT_Unorm_8_8_8_8, TEXT("GBufferCelshading"), false, true, true, true);
	
	TargetInfrared = 7;
	Info.Targets[7].Init(GBT_Unorm_8_8_8_8, TEXT("TargetInfrared"), false, true, true, true);

	TargetSeparatedMainDirLight = 8; // 5
	if (Params.bHasPrecShadowFactor)
	{
		TargetGBufferE = 8; // 6
		Info.Targets[8].Init(GBT_Unorm_8_8_8_8, TEXT("GBufferE"), false, true, true, true); // 6
		TargetSeparatedMainDirLight = 9; // 7
	}
}
else
{
	// should never hit this path
	check(0);
}

Info.Slots[GBS_Infrared] = FGBufferItem(GBS_Infrared, GBC_Raw_Unorm_8_8_8_8, GBCH_Both);
Info.Slots[GBS_Infrared].Packing[0] = FGBufferPacking(TargetInfrared, 0, 0);
Info.Slots[GBS_Infrared].Packing[1] = FGBufferPacking(TargetInfrared, 1, 1);
Info.Slots[GBS_Infrared].Packing[2] = FGBufferPacking(TargetInfrared, 2, 2);
Info.Slots[GBS_Infrared].Packing[3] = FGBufferPacking(TargetInfrared, 3, 3);
```

- **GBufferInfo.h** 可以根据需要设置 MaxTargets
- **ShaderGenerationUtil.cpp** 打开对应的材质接口

```C++
static void SetSlotsForShadingModelType(bool Slots[], EMaterialShadingModel ShadingModel, bool bMergeCustom)
{
	switch (ShadingModel)
	{
	case MSM_Unlit:
		Slots[GBS_SceneColor] = true;
		break;
	case MSM_DefaultLit:
		SetSharedGBufferSlots(Slots);
		break;
	case MSM_Subsurface:
		SetSharedGBufferSlots(Slots);
		if (bMergeCustom)
		{
			Slots[GBS_CustomData] = true;
		}
		else
		{
			Slots[GBS_SubsurfaceColor] = true;
			Slots[GBS_Opacity] = true;
		}
		break;
	case MSM_PreintegratedSkin:
		SetSharedGBufferSlots(Slots);
		if (bMergeCustom)
		{
			Slots[GBS_CustomData] = true;
		}
		else
		{
			Slots[GBS_SubsurfaceColor] = true;
			Slots[GBS_Opacity] = true;
		}
		break;
	case MSM_ClearCoat:
		SetSharedGBufferSlots(Slots);
		if (bMergeCustom)
		{
			Slots[GBS_CustomData] = true;
		}
		else
		{
			Slots[GBS_ClearCoat] = true;
			Slots[GBS_ClearCoatRoughness] = true;
		}
		break;
	case MSM_SubsurfaceProfile:
		SetSharedGBufferSlots(Slots);
		if (bMergeCustom)
		{
			Slots[GBS_CustomData] = true;
		}
		else
		{
			Slots[GBS_SubsurfaceProfile] = true;
			Slots[GBS_Opacity] = true;
		}
		break;
	case MSM_TwoSidedFoliage:
		SetSharedGBufferSlots(Slots);
		if (bMergeCustom)
		{
			Slots[GBS_CustomData] = true;
		}
		else
		{
			Slots[GBS_SubsurfaceColor] = true;
			Slots[GBS_Opacity] = true;
		}
		break;
	case MSM_Hair:
		SetSharedGBufferSlots(Slots);
		if (bMergeCustom)
		{
			Slots[GBS_CustomData] = true;
		}
		else
		{
			Slots[GBS_HairSecondaryWorldNormal] = true;
			Slots[GBS_HairBacklit] = true;
		}
		break;
	case MSM_Cloth:
		SetSharedGBufferSlots(Slots);
		if (bMergeCustom)
		{
			Slots[GBS_CustomData] = true;
		}
		else
		{
			Slots[GBS_SubsurfaceColor] = true;
			Slots[GBS_Cloth] = true;
		}
		break;
	case MSM_Eye:
		SetSharedGBufferSlots(Slots);
		if (bMergeCustom)
		{
			Slots[GBS_CustomData] = true;
		}
		else
		{
			Slots[GBS_SubsurfaceProfile] = true;
			Slots[GBS_IrisNormal] = true;
			Slots[GBS_Opacity] = true;
		}
		break;
	case MSM_SingleLayerWater:
		SetSharedGBufferSlots(Slots);
		Slots[GBS_SeparatedMainDirLight] = true;
		break;
	case MSM_ThinTranslucent:
		// thin translucent doesn't write to the GBuffer
		break;

	case MSM_CS_Default:
		SetSharedGBufferSlots(Slots);
		//#IF USE_NEW_GBUFFER
		Slots[GBS_Celshading] = true;
		//#ELSE
		Slots[GBS_CustomData] = true;
		//#ENDIF
			break;
	case MSM_CS_Subsurface:
		SetSharedGBufferSlots(Slots);
		//#IF USE_NEW_GBUFFER
		Slots[GBS_Celshading] = true;
		//#ENDIF
		if (bMergeCustom)
		{
			Slots[GBS_CustomData] = true;
		}
		else
		{
			Slots[GBS_SubsurfaceColor] = true;
			Slots[GBS_Opacity] = true;
		}
		break;
	case MSM_Infrared_Default:
		SetSharedGBufferSlots(Slots);
		//#IF USE_NEW_GBUFFER
		Slots[GBS_Celshading] = true;
		Slots[GBS_Infrared] = true;
		//#ENDIF
		if (bMergeCustom)
		{
			Slots[GBS_CustomData] = true;
		}
		else
		{
			Slots[GBS_SubsurfaceColor] = true;
			Slots[GBS_Opacity] = true;
		}
		break;

	}
}
```

### HLSL 

- **DeferredShadingCommon.ush** 声明对应的 GBuffer

```HLSL
// Matches FSceneTextureParameters
Texture2D SceneDepthTexture;
Texture2D<uint2> SceneStencilTexture;
Texture2D GBufferATexture;
Texture2D GBufferBTexture;
Texture2D GBufferCTexture;
Texture2D GBufferDTexture;
Texture2D GBufferETexture;
Texture2D GBufferVelocityTexture;
Texture2D GBufferFTexture;
Texture2D GBufferCelshadingTexture; // Celshading Lucas: create a global texture2D
Texture2D GBufferInfraredTexture; // Celshading Lucas: create a global texture2D
Texture2D<uint> SceneLightingChannels;

#define GBufferInfraredTextureSampler GlobalPointClampedSampler // Celshading Lucas: Define a new point sampler

struct FGBufferData
{
	// normalized
	half3 WorldNormal;
	// normalized, only valid if HAS_ANISOTROPY_MASK in SelectiveOutputMask
	half3 WorldTangent;
	// 0..1 (derived from BaseColor, Metalness, Specular)
	half3 DiffuseColor;
	// 0..1 (derived from BaseColor, Metalness, Specular)
	half3 SpecularColor;
	// 0..1, white for SHADINGMODELID_SUBSURFACE_PROFILE and SHADINGMODELID_EYE (apply BaseColor after scattering is more correct and less blurry)
	half3 BaseColor;
	// 0..1
	half Metallic;
	// 0..1
	half Specular;
	// 0..1
	half4 CustomData;
	// AO utility value
	half GenericAO;
	// Indirect irradiance luma
	half IndirectIrradiance;
	// Static shadow factors for channels assigned by Lightmass
	// Lights using static shadowing will pick up the appropriate channel in their deferred pass
	half4 PrecomputedShadowFactors;
	// 0..1
	half Roughness;
	// -1..1, only valid if only valid if HAS_ANISOTROPY_MASK in SelectiveOutputMask
	half Anisotropy;
	// 0..1 ambient occlusion  e.g.SSAO, wet surface mask, skylight mask, ...
	half GBufferAO;
	// Bit mask for occlusion of the diffuse indirect samples
	uint DiffuseIndirectSampleOcclusion;
	// 0..255 
	uint ShadingModelID;
	// 0..255 
	uint SelectiveOutputMask;
	// 0..1, 2 bits, use CastContactShadow(GBuffer) or HasDynamicIndirectShadowCasterRepresentation(GBuffer) to extract
	half PerObjectGBufferData;
	// in world units
	half CustomDepth;
	// Custom depth stencil value
	uint CustomStencil;
	// in unreal units (linear), can be used to reconstruct world position,
	// only valid when decoding the GBuffer as the value gets reconstructed from the Z buffer
	half Depth;
	// Velocity for motion blur (only used when WRITES_VELOCITY_TO_GBUFFER is enabled)
	half4 Velocity;

	// 0..1, only needed by SHADINGMODELID_SUBSURFACE_PROFILE and SHADINGMODELID_EYE which apply BaseColor later
	half3 StoredBaseColor;
	// 0..1, only needed by SHADINGMODELID_SUBSURFACE_PROFILE and SHADINGMODELID_EYE which apply Specular later
	half StoredSpecular;
	// 0..1, only needed by SHADINGMODELID_EYE which encodes Iris Distance inside Metallic
	half StoredMetallic;

	// Curvature for mobile subsurface profile
	half Curvature;

	// 0..1, Celshading Lucas: Celshading values
	float4 Celshading;

	float4 Infrared;
};
```

- **SceneTexturesCommon.ush** 声明 sampler

```HLSL
#define SceneTexturesStruct_GBufferInfraredTextureSampler SceneTexturesStruct.PointClampSampler // Celshading Lucas: another sampler for scene texture
```

- **DeferredDecal.usf** 添加默认值

```HLSL
	GBufferData.Infrared = 0; // Celshading Lucas : dummy value for decals
```

- **Common.ush** 可以设置大小 FPixelShaderOut
- **ShaderOutputCommon.ush PixelShaderOutputCommon.ush** 添加 MRT 输出

```HLSL
// ShaderOutputCommon.ush
 
#ifndef PIXELSHADEROUTPUT_MRT5
	#define PIXELSHADEROUTPUT_MRT5 0
#endif
 
#ifndef PIXELSHADEROUTPUT_MRT6
	#define PIXELSHADEROUTPUT_MRT6 0
#endif
 
#ifndef PIXELSHADEROUTPUT_MRT7
	#define PIXELSHADEROUTPUT_MRT7 0
#endif
 
 
// PixelShaderOutputCommon.ush
 
PIXELSHADER_EARLYDEPTHSTENCIL
void MainPS
	(
#if PIXELSHADEROUTPUT_INTERPOLANTS || PIXELSHADEROUTPUT_BASEPASS
#if IS_NANITE_PASS
		FNaniteFullscreenVSToPS NaniteInterpolants,
#else
		FVertexFactoryInterpolantsVSToPS Interpolants,
#endif
#endif
#if PIXELSHADEROUTPUT_BASEPASS
		FBasePassInterpolantsVSToPS BasePassInterpolants,
#elif PIXELSHADEROUTPUT_MESHDECALPASS
		FMeshDecalInterpolants MeshDecalInterpolants,
#endif
 
// [...]
 
#if PIXELSHADEROUTPUT_MRT5
		, out float4 OutTarget5 : SV_Target5
#endif
 
#if PIXELSHADEROUTPUT_MRT6
		, out float4 OutTarget6 : SV_Target6
#endif
 
#if PIXELSHADEROUTPUT_MRT7
		, out float4 OutTarget7 : SV_Target7
#endif
 
// [...]
 
{
 
	// ---------------------------------------------------------------------------------
#if IS_NANITE_PASS && (PIXELSHADEROUTPUT_INTERPOLANTS || PIXELSHADEROUTPUT_BASEPASS)
	FVertexFactoryInterpolantsVSToPS Interpolants = (FVertexFactoryInterpolantsVSToPS)0;
	Interpolants.ViewIndex = NaniteInterpolants.ViewIndex;
#endif
 
	FPixelShaderIn PixelShaderIn = (FPixelShaderIn)0;
	FPixelShaderOut PixelShaderOut = (FPixelShaderOut)0;
 
// [...]
 
#if PIXELSHADEROUTPUT_MRT5
	OutTarget5 = PixelShaderOut.MRT[5];
#endif
 
#if PIXELSHADEROUTPUT_MRT6
	OutTarget6 = PixelShaderOut.MRT[6];
#endif
 
#if PIXELSHADEROUTPUT_MRT7
	OutTarget7 = PixelShaderOut.MRT[7];
#endif
 
// [...]
 
}
```

### Shading model 绑定 slot

- **ShaderGenerationUtil.cpp** 

```C++
if (Mat.MATERIAL_SHADINGMODEL_INFRARED_DEFAULT)
{
	SetStandardGBufferSlots(Slots, bWriteEmissive, bHasTangent, bHasVelocity, bHasStaticLighting, bIsStrataMaterial);
	Slots[GBS_CustomData] = bUseCustomData;
	//#IF USE_NEW_GBUFFER
	Slots[GBS_Celshading] = true;
	Slots[GBS_Infrared] = true;
	//#ENDIF
}

case MSM_Infrared_Default:
	SetSharedGBufferSlots(Slots);
	//#IF USE_NEW_GBUFFER
	Slots[GBS_Celshading] = true;
	Slots[GBS_Infrared] = true;
	//#ENDIF
	if (bMergeCustom)
	{
		Slots[GBS_CustomData] = true;
	}
	else
	{
		Slots[GBS_SubsurfaceColor] = true;
		Slots[GBS_Opacity] = true;
	}
	break;
```

### 填充 FGBufferData

- **ShadingModelsMaterial.ush** 添加写入，不过这里应该是只针对 basepass 的修改

```C++
#if MATERIAL_SHADINGMODEL_INFRARED_DEFAULT
	else if (ShadingModel == SHADINGMODELID_INFRARED_DEFAULT)
	{
		GBuffer.Celshading.x = 0.95f;
		GBuffer.Celshading.y = 0.25f;
		GBuffer.Celshading.z = 0.35f;
		GBuffer.Celshading.w = 0.45f;

		GBuffer.Infrared.r = GetMaterialCustomData0(MaterialParameters) / 2000.f;
		GBuffer.Infrared.g = GetMaterialCustomData0(MaterialParameters);
		GBuffer.Infrared.b = GetCelShadingSelection0(MaterialParameters);
		GBuffer.Infrared.a = Opacity;

		GBuffer.CustomData.r = GetMaterialCustomData0(MaterialParameters) / 5000.f;
		GBuffer.CustomData.g = GetMaterialCustomData1(MaterialParameters);
		GBuffer.CustomData.b = GetCelShadingSelection0(MaterialParameters);
		GBuffer.CustomData.a = Opacity;
	}
#endif
```

- git 设置代理

```shell
git config --global https.proxy http://127.0.0.1:1080

git config --global https.proxy https://127.0.0.1:1080

git config --global --unset http.proxy

git config --global --unset https.proxy

git config --global http.proxy socks5://127.0.0.1:7890
git config --global https.proxy socks5://127.0.0.1:7890
git config --global http.https://github.com.proxy socks5://127.0.0.1:7890
```

## 小结

花了整整两周来修改和调试，期间遇到不少问题，有的问题被解决得也莫名其妙，好在终于有个能用的版本了，打算新写一篇用一个全新的 UE 来改关键的内容，希望别再出什么奇奇怪怪的问题了

## References

- [New shading models and changing the GBuffer](https://dev.epicgames.com/community/learning/tutorials/2R5x/unreal-engine-new-shading-models-and-changing-the-gbuffer?locale=ru-ru)
- [Adding a new Shading Model in UE5](https://zhuanlan.zhihu.com/p/553585780)
- [Pimpl技术——编译期封装](https://www.cnblogs.com/KillerAery/p/9539705.html)
- [UE(1)：材质系统](https://cloud.tencent.com/developer/article/2197771)
- [UE5【实践】4.外描边OutLinePass及材质参数传递（含材质实例）](https://zhuanlan.zhihu.com/p/576774695)
- [UE5 Add Custom Variables in Material](https://zhuanlan.zhihu.com/p/565776677)
- [Customize GBuffer In UE5](https://zhuanlan.zhihu.com/p/568775542)