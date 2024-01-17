---
title: "Minecraft Fabric模组开发教程#11 添加流体"
date: 2024-01-15T15:45:08+08:00
---

## 前言

本教程将会教你如何添加流体，什么是流体呢？就是像水、熔岩这样的东西，我们将会添加一种新的流体。

## 创建流体

在原版中流体都继承了`net.minecraft.fluid.FlowableFluid`，所以我们也继承它。

首先创建一个抽象类`AwesomeFluid`，然后继承`FlowableFluid`，然后实现一些方法。

- `matchesType`：判断流体是否是我们的流体
- `isInfinite`：是否是无限流体，就是像水一样，无限多
- `beforeBreakingBlock`：当流体破坏方块时，会调用这个方法，我们在这里把方块掉落，比如火把被水破坏时，会掉落火把
- `canBeReplacedWith`：是否可以被替换
- `getFlowSpeed`：流体流动速度
- `getLevelDecreasePerBlock`：水返回 1，熔岩在主世界时返回 2 并且在下界时返回 1
- `getTickRate`：水返回 5，熔岩在主世界时返回 30 并且在下界时返回 10
- `getBlastResistance`：爆炸抗性，水和熔岩都是 100

```java
public abstract class AwesomeFluid extends FlowableFluid {
    @Override
    public boolean matchesType(Fluid fluid) {
        return fluid == getStill() || fluid == getFlowing();
    }

    @Override
    protected boolean isInfinite(World world) {
        return true;
    }

    @Override
    protected void beforeBreakingBlock(WorldAccess world, BlockPos pos, BlockState state) {
        final BlockEntity blockEntity = state.hasBlockEntity() ? world.getBlockEntity(pos) : null;
        Block.dropStacks(state, world, pos, blockEntity);
    }

    @Override
    protected boolean canBeReplacedWith(FluidState state, BlockView world, BlockPos pos, Fluid fluid, Direction direction) {
        return false;
    }

    @Override
    protected int getFlowSpeed(WorldView world) {
        return 4;
    }

    @Override
    protected int getLevelDecreasePerBlock(WorldView world) {
        return 1;
    }

    @Override
    public int getTickRate(WorldView world) {
        return 5;
    }

    @Override
    protected float getBlastResistance() {
        return 100;
    }
}
```

之后我们创建一个`EndWaterFluid`枚举类，继承`AwesomeFluid`，然后实现一些方法，这里先返回`null`，我们之后会修改。

```java
public abstract class EndWaterFluid extends AwesomeFluid {
    @Override
    public Fluid getStill() {
        return null;
    }

    @Override
    public Fluid getFlowing() {
        return null;
    }

    @Override
    public Item getBucketItem() {
        return null;
    }

    @Override
    protected BlockState toBlockState(FluidState state) {
        return null;
    }
}
```

接着我们在`EndWaterFluid`类中创建一个`Flowing`和`Still`类，继承`EndWaterFluid`，然后实现一些方法。

- `getStill`：是否是静止流体
- `getLevel`：流体的等级
- `appendProperties`：添加属性

```java
public static class Flowing extends EndWaterFluid {
    @Override
    public boolean isStill(FluidState state) {
        return false;
    }

    @Override
    public int getLevel(FluidState state) {
        return state.get(LEVEL);
    }

    @Override
    protected void appendProperties(StateManager.Builder<Fluid, FluidState> builder) {
        super.appendProperties(builder);
        builder.add(LEVEL);
    }
}
```

- `getStill`：是否是静止流体
- `getLevel`：流体的等级

```java
public static class Still extends EndWaterFluid {
    @Override
    public boolean isStill(FluidState state) {
        return true;
    }

    @Override
    public int getLevel(FluidState state) {
        return 8;
    }
}
```

## 注册流体

分别注册静止流体和流动流体，然后注册流体桶，最后注册流体方块，这里我们使用`FabricBlockSettings.copy`复制水的属性。

```java
public static final FlowableFluid STILL_END_WATER = Registry.register(Registries.FLUID, new Identifier("awesome", "end_water"), new EndWaterFluid.Still());
public static final FlowableFluid FLOWING_END_WATER = Registry.register(Registries.FLUID, new Identifier("awesome", "flowing_end_water"), new EndWaterFluid.Flowing());
public static final Item END_WATER_BUCKET = Registry.register(Registries.ITEM, new Identifier("awesome", "end_water_bucket"), new BucketItem(STILL_END_WATER, new FabricItemSettings().recipeRemainder(Items.BUCKET).maxCount(1)));
public static final Block END_WATER = Registry.register(Registries.BLOCK, new Identifier("awesome", "end_water"), new FluidBlock(STILL_END_WATER, FabricBlockSettings.copy(Blocks.WATER)));
```

最后补全`EndWaterFluid`类。

```java
public abstract class EndWaterFluid extends AwesomeFluid {
    @Override
    public Fluid getStill() {
        return ExampleMod.STILL_END_WATER;
    }

    @Override
    public Fluid getFlowing() {
        return ExampleMod.FLOWING_END_WATER;
    }

    @Override
    public Item getBucketItem() {
        return ExampleMod.END_WATER_BUCKET;
    }

    @Override
    protected BlockState toBlockState(FluidState state) {
        return ExampleMod.END_WATER.getDefaultState().with(FluidBlock.LEVEL, WaterFluid.getBlockStateLevel(state));
    }
}
```

## 渲染设置

在`onInitialize`设置流体渲染。

静止流体和流动流体的纹理都是`minecraft:block/water_still`，`minecraft:block/water_flow`，颜色是`0x730D95`。

```java
FluidRenderHandlerRegistry.INSTANCE.register(STILL_END_WATER, FLOWING_END_WATER, new SimpleFluidRenderHandler(
            new Identifier("minecraft:block/water_still"),
            new Identifier("minecraft:block/water_flow"),
            0x730D95
));

BlockRenderLayerMap.INSTANCE.putFluids(RenderLayer.getTranslucent(), STILL_END_WATER, FLOWING_END_WATER);
```

## 流体桶纹理

![end_water_bucket](/assets/fabric2024/end_water_bucket.png)

## 流体生成

和矿物生成差不多，都需要在`configured_feature`和`placed_feature`中创建配置。

我们找到原版游戏中的`data/minecraft/worldgen/configured_feature/spring_water.json`和`data/minecraft/worldgen/placed_feature/spring_water.json`，然后将其修改为我们的配置。

首先在`configured_feature`中创建，`end_water.json`。

```json
{
  "type": "minecraft:spring_feature",
  "config": {
    "hole_count": 1,
    "requires_block_below": true,
    "rock_count": 4,
    "state": {
      "Name": "awesome:end_water",
      "Properties": {
        "falling": "true"
      }
    },
    "valid_blocks": [
      "minecraft:stone",
      "minecraft:granite",
      "minecraft:diorite",
      "minecraft:andesite",
      "minecraft:deepslate",
      "minecraft:tuff",
      "minecraft:calcite",
      "minecraft:dirt",
      "minecraft:snow_block",
      "minecraft:powder_snow",
      "minecraft:packed_ice"
    ]
  }
}
```

之后在`placed_feature`中创建，`end_water.json`。

```json
{
  "feature": "awesome:end_water",
  "placement": [
    {
      "type": "minecraft:count",
      "count": 25
    },
    {
      "type": "minecraft:in_square"
    },
    {
      "type": "minecraft:height_range",
      "height": {
        "type": "minecraft:uniform",
        "max_inclusive": {
          "absolute": 192
        },
        "min_inclusive": {
          "above_bottom": 0
        }
      }
    },
    {
      "type": "minecraft:biome"
    }
  ]
}
```

最后注册`PlacedFeature`。

```java
public static final RegistryKey<PlacedFeature> END_WATER_PLACED_FEATURE = RegistryKey.of(RegistryKeys.PLACED_FEATURE, new Identifier("awesome", "end_water"));
```

并且在`BiomeModifications`中注册`END_WATER_PLACED_FEATURE`。

```java
BiomeModifications.addFeature(BiomeSelectors.foundInOverworld(), GenerationStep.Feature.FLUID_SPRINGS, END_WATER_PLACED_FEATURE);
```

![11-1](/assets/fabric2024/11-1.png)