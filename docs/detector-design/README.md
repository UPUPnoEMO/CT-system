# CT 探测器设计 · Detector Design

> CT 探测器的核心是将 X 射线转换为可测量的电信号，涉及闪烁体/半导体材料、ASIC 读出、机械封装等关键环节。

---

## 1. 探测器材料

### 1.1 CdTe 和 CdZnTe（CZT）半导体探测器

#### 材料优势

- **直接能量转换**：CdTe 和 CZT 探测器能将吸收的 γ 射线能量直接转换为电信号，具有能量分辨和光子计数能力。
- **高探测效率**：高原子序数和密度提供出色的射线阻止能力，适用于高灵敏度探测。
- **室温操作**：宽禁带宽度（CdTe 1.5 eV，CZT 1.572 eV）允许室温工作，无需冷却系统。
- **低漏电流**：半绝缘特性可实现大耗尽深度（毫米至厘米级）和低漏电流（pA 至 nA 级）。
- **良好电荷传输**：适中的载流子迁移率和寿命支持有效的电荷收集。
- **可像素化**：化学和机械特性支持制造紧凑型成像设备。

#### 医疗应用场景

| 应用 | 说明 |
|------|------|
| 核医学 SPECT / 分子乳腺成像（MBI） | 利用优异能量分辨率（140 keV 下 2-5% FWHM）提高散射抑制和空间分辨率 |
| 骨密度测量（DEXA） | 双能 X 射线吸收测量，利用高探测效率和能量灵敏度 |
| 数字放射成像 | 光子计数技术提升信噪比和对比度分辨率；直接转换探测器提供高空间分辨率 |
| 计算机断层扫描（CT） | 多能 CT 是重要挑战，需探测器在高速光子流下快速响应 |
| 紧凑型设备 | 前列腺成像和结肠癌检测的微型胶囊相机 |

#### CdTe vs CZT 材料特性对比

| 特性 | CdTe | CZT |
|------|-------|-----|
| 禁带宽度 | 1.5 eV | 1.572 eV（含 ~10% Zn）|
| 电阻率 | ~2×10¹⁰ Ω·cm | ~5×10¹⁰ Ω·cm |
| 导电类型 | p 型（掺 Cd）| n 型（掺 In/Ga/Al）|
| 肖特基接触 | In/Al | Pt/Au |
| 漏电流 | 较低 | 较高 |
| 晶格强度 | 基础 | Zn 固溶强化，更脆 |
| Zn 分凝系数 | — | ~1.3（轴向浓度不均）|

**总结**：CZT 通过 Zn 合金化提升电阻率和机械强度，但晶体生长和均匀性控制更具挑战；CdTe 工艺更成熟，漏电流更低。

#### 主要技术困难

1. **材料缺陷控制**：点缺陷（如 Cd 空位）、位错、沉淀物等导致载流子捕获和复合，影响电荷传输和能量分辨率。
2. **晶体生长挑战**：寄生成核导致多晶形成；热应力、组分偏析（Zn 分布不均）引入应力缺陷。
3. **电学补偿复杂性**：通过掺杂（In、Cl 等）补偿本征缺陷，但难以同时实现高电阻率和高载流子传输。
4. **高通量应用限制**：高光子通量下极化效应、电荷堆积和漏电流增加，影响 CT 等应用的稳定性和速度。

#### 未来发展方向

- **材料优化**：改进晶体生长技术（THM、VB 法），减少缺陷密度；开发新型掺杂策略。
- **探测器设计创新**：像素化探测器 + 3D 信号校正；高速读出 ASIC（>10⁷ counts/s/mm²）。
- **多模态集成**：与 MRI、PET 结合，实现多功能成像。

---

## 2. 探测器信号链与电荷换算

### 2.1 基本原理

CT 探测器将 X 射线转换为数字图像的过程：

```
X 射线 → 闪烁体/半导体 → 可见光/电荷 → ASIC 读出 → 数字信号（DN 值）
```

### 2.2 关键参数与换算关系

| 参数 | 符号 | 数值/说明 |
|------|------|-----------|
| 图像尺寸 | — | 36×24 像素 |
| 图像存储格式 | — | 8 bit（0~255）|
| 图像灰度范围 | — | 2⁸ = 256 级 |
| 满量程电荷 | FSR | 8.9 nC（本例）|
| 刻度因子 | K | FSR / 255 = 8.9/255 |
| DN 最大值（24bit）| MAX_DN | 2²⁴-1 = 16,777,215 |
| 满量程电荷（24bit）| — | 29 pC |
| 单个电子电荷量 | e | 1.602×10⁻¹⁹ C |

### 2.3 换算公式

```
DN → 电荷量 Q：
  Q = DN × (FSR / MAX_DN)

电荷量 Q → 电子数目：
  e_num = Q / e = Q / 1.602×10⁻¹⁹

DN → 电流（安培）：
  I = (DN × CHARGE_PER_DN) / Δt
  其中 CHARGE_PER_DN = FULL_SCALE_CHARGE / MAX_DN
```

### 2.4 Python 计算工具

```python
import numpy as np

# 系统常量定义
MAX_DN = 2**24 - 1          # 24bit 最大 DN 值：16777215
FULL_SCALE_CHARGE = 29e-12   # 满量程电荷：29 pC
ELECTRON_CHARGE = 1.602e-19  # 单个电子电荷量（库仑）
CHARGE_PER_DN = FULL_SCALE_CHARGE / MAX_DN  # 每个 DN 对应的电荷量

def electrons_to_fC(electrons):
    """将电子数目转换为飞库仑(fC)"""
    return electrons * 0.0001602

def electrons_to_DN(electron_count):
    """将电子数目转换为 DN 值"""
    charge = electron_count * ELECTRON_CHARGE
    return charge / CHARGE_PER_DN

def calculate_current_from_DN(delta_DN, delta_time_sec):
    """从 DN 值变化计算电流（安培）"""
    total_charge = CHARGE_PER_DN * delta_DN
    return total_charge / delta_time_sec

# 漏电流计算示例
# DN1=68764 (347ns), DN2=69177 (694ns)
delta_DN = 69177 - 68764
delta_time_sec = (694 - 347) * 1e-9
current_A = calculate_current_from_DN(delta_DN, delta_time_sec)
print(f"漏电流 = {current_A:.2e} A = {current_A*1e12:.2f} pA")
```

### 2.5 漏电流计算实例

| 参数 | 值 |
|------|-----|
| DN₁ | 68764 |
| 积分时间₁ | 347 ns |
| DN₂ | 69177 |
| 积分时间₂ | 694 ns |
| ΔDN | 413 |
| Δt | 347 ns |
| **漏电流** | **约 34.5 pA** |

---

## 3. 探测器封装与可靠性

> 详见 [frontend-process/](../frontend-process/)（前段工艺）及 [reliability-analysis/](../reliability-analysis/)（可靠性分析）

---

## 参考资源

- **Geant4 仿真**：参见 [geant4-simulation/](../geant4-simulation/) — 能谱仿真与探测器响应建模
- **ASIC 设计**：参见 [asic-design/](../asic-design/) — 读出芯片设计
- **质量控制**：参见 [quality-control/](../quality-control/) — 探测器 QC 全流程
