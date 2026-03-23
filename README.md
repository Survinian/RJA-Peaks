# RJA-Peaks · 北京周边山峰多库综合地图

一个基于 MapTiler SDK 的交互式山峰地形显著度地图，整合了北京及周边多个区域的山峰数据库，支持按高差、突起度（Jut）、地形锐度（ATS）等多维指标进行可视化与筛选。

**在线访问：** [https://Survinian.github.io/RJA-Peaks/](https://Survinian.github.io/RJA-Peaks/)

---

## 数据库说明

### 基础库

| 库名 | 覆盖范围 | 来源 |
|------|---------|------|
| 北京周边库 | 北京市域及少量周边区域 | 个人标注 |
| 百里溪库 | 京津冀 | 前辈**百里溪**系列文章，特此致谢 🙏 |

### 精英山峰库

精英库山峰通过 GEE + Python 多步自动筛选生成，覆盖以下区域：

| 库名 | 经度范围 | 纬度范围 | 标注色 |
|------|---------|---------|------|
| 北京周边精英库 | 114.5°E–118.0°E | 39.0°N–41.2°N | 红 |
| 冀东精英库 | 118.0°E–121.5°E | 39.0°N–41.2°N | 橙 |
| 宣大精英库 | 113.0°E–114.5°E | 39.0°N–41.2°N | 棕 |
| 中太行精英库 | 113.0°E–114.5°E | 37.0°N–39.0°N | 粉红 |
| 南太行精英库 | 113.0°E–114.5°E | 35.0°N–37.0°N | 橙红 |
| 燕坝精英库 | 114.5°E–118.0°E | 41.2°N–43.2°N | 绿 |
| 察哈尔精英库 | 113.0°E–114.5°E | 41.2°N–43.2°N | 深青 |
| 赤峰精英库 | 118.0°E–122.0°E | 41.2°N–43.2°N | 紫灰 |
| 绥朔精英库 | 111.0°E–113.0°E | 39.0°N–41.2°N | 红灰 |
| 吕梁山精英库 | 111.0°E–113.0°E | 37.0°N–39.0°N | 青蓝 |
| 汾沁精英库 | 111.0°E–113.0°E | 34.6°N–37.0°N | 橙灰 |
| 泰沂精英库 | 116.0°E–119.6°E | 34.8°N–37.0°N | 紫 |
| 胶东精英库 | 119.6°E–122.8°E | 35.5°N–38.5°N | 蓝 |

精英库生成流程：
1. **GEE 提取**（`_relief_map_without_input.js`）：基于 JAXA ALOS AW3D30 V4.1 高程模型，在各研究区域内动态提取局部极大值点，计算各采样半径下的高差排名，按显著性阈值初步过滤
2. **Python 筛选**（`_unname_peaks_statistics.py`）：与参考库（北京周边库 / 百里溪库）比对，进行命名匹配与初步质量过滤
3. **Python 精英筛选**（`_generate_regional_elite.py`）：三步筛选——库内排名（Step 1）、绝对阈值验证（Step 2）、外卡通道（Step 3，前 1% / 5% / 10% 直通）

---

## 指标体系

所有指标在多个采样半径（125 m–10,000 m）下独立计算，形成完整的多尺度剖面。

### 第一阶：基础起伏场

**高差（Relief）**

$$R_{mean}(r) = \frac{1}{N_r} \sum_{i \in D_r}(Z_{peak} - Z_i)$$

$$R_{max}(r) = \max_{i \in D_r}(Z_{peak} - Z_i)$$

**突起度（Jut）**：高差与仰角的乘积，保留负值以惩罚卫峰和台地

$$J_{mean}(r) = \frac{1}{N_r} \sum_{i \in D_r} \frac{(Z_{peak}-Z_i)\cdot|Z_{peak}-Z_i|}{\sqrt{(Z_{peak}-Z_i)^2+d_i^2}}$$

$$J_{max}(r) = \max_{i \in D_r} \frac{(Z_{peak}-Z_i)\cdot|Z_{peak}-Z_i|}{\sqrt{(Z_{peak}-Z_i)^2+d_i^2}}$$

### 第二阶：无量纲形态因子

| 指标 | 公式 | 物理意义 |
|------|------|---------|
| 极限绝壁度 CI | $J_{max} / R_{max}$ | 最陡剖面的 sin(θ)，越接近 1 越像垂直断崖 |
| 全局陡峭度 GS | $J_{mean} / R_{max}$ | 满视野平均压迫感占极限高差的比例 |
| 山体纤细度 SL | $R_{mean} / R_{max}$ | 山体骨感程度，接近 1 为细针，接近 0 为平顶高原 |
| 尖锥各向同性 SF | $J_{mean} / J_{max}$ | 险峻是否全方位，接近 1 为孤立尖锥，接近 0 为单面绝壁 |

### 第三阶：综合地形锐度

**纯形态锐度指数（FSI）**

$$FSI(r) = \frac{J_{mean}(r) \cdot R_{mean}(r)}{[R_{max}(r)]^2}$$

**绝对地形锐度（ATS）**：本体系核心指标，单位为米

$$ATS(r) = \frac{J_{mean}(r) \cdot R_{mean}(r)}{R_{max}(r)}$$

ATS 可理解为：将这座山的视觉压迫感等效为一根立在平地上的垂直定海神针，这根针有多高。

### 第四阶：等效地形尺度

| 指标 | 公式 | 物理意义 |
|------|---------|---------|
| 等效突出半径 $R_{prominence}$ | $\arg\max_r J_{mean}(r)$ | 山体骨架的边界，越过此界平缓基座开始主导 |
| 等效基座半径 $R_{base}$ | $\arg\max_r ATS(r)$ | 全山领地的终极边界，客观界定山的底面积 |

---

## 综合击败率指标

每座山峰在库内的综合排名由以下五项击败率的均值决定：

| 指标 | 字段 | 含义 |
|------|------|------|
| 高差总体表现 | `mean_relief_crp` | mean_relief 在所有采样半径下的均值排名 |
| 平均Jut总体表现 | `mean_jut_crp` | mean_jut 在所有采样半径下的均值排名 |
| ATS总体表现 | `mean_ats_crp` | ATS 在所有采样半径下的均值排名 |
| 最大Jut峰值表现 | `max_jut_crp` | max_jut 在所有采样半径下的最大值排名 |
| 最大ATS峰值 | `max_ats_crp` | ATS 在所有采样半径下的最大值排名 |

击败率 = 1 − 排名百分位，越高越优秀（0% 为最差，100% 为最优）。

---

## 地图功能

- **多库叠加显示**：可独立开关各数据库图层
- **山峰点击弹窗**：显示完整指标剖面、各距离段排名、高差 / Jut / ATS 随半径变化曲线
- **多维筛选**：五项综合击败率筛除（取交集），满足所有条件才显示
- **距离段筛选**：按极短距离至超长距离七个距离段独立筛选
- **等效半径圆**：可视化每座山峰的等效突出半径和等效基座半径范围
- **山峰检索**：支持按名称模糊搜索定位
- **3D地形**：基于 MapTiler 地形数据的三维山体渲染

---

## 文件结构

```
index.html                                 主地图文件
peaks/
  output_beijing_geojson/                  北京周边库
  output_bailixi_geojson/                  百里溪库
  output_beijing_around_elites_geojson/    北京周边精英库
  output_jidong_elites_geojson/            冀东精英库
  output_xuanda_elites_geojson/            宣大精英库
  output_zhongtaihang_elites_geojson/      中太行精英库
  output_nantaihang_elites_geojson/        南太行精英库
  output_chahar_elites_geojson/            察哈尔精英库
  output_yanba_elites_geojson/             燕坝精英库
  output_chifeng_elites_geojson/           赤峰精英库
  output_suishuo_elites_geojson/           绥朔精英库
  output_lvliangshan_elites_geojson/       吕梁山精英库
  output_fenqin_elites_geojson/            汾沁精英库
  output_taiyi_elites_geojson/             泰沂精英库
  output_jiaodong_elites_geojson/          胶东精英库
```

每个 `output_*` 目录包含：
- `peaks_*.geojson` — 山峰主点层
- `*_jut_bases.geojson` — 最优 Jut 观测点
- `*_jut_profiles.json` — 各半径剖面数据
- `*_epr_circles.geojson` — 等效突出半径圆
- `*_base_circles.geojson` — 等效基座半径圆

---

## 致谢

- 百里溪库数据来源于前辈**百里溪**多年积累的系列山峰记录文章，感谢其无私分享。
- 高程数据来源：[JAXA ALOS AW3D30 V4.1](https://www.eorc.jaxa.jp/ALOS/en/dataset/aw3d30/aw3d30_e.htm)
- 地图底图及地形数据：[MapTiler](https://www.maptiler.com/)
