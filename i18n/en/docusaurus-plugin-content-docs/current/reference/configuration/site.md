---
sidebar_position: 2
---

# `site`

Describes the location with the existing and required devices of the home installation and is responsible for regulating the available power.

To regulate charging with PV surplus, a readable meter directly behind the grid connection point is necessary. In addition, devices for PV power and house battery(ies) can also be specified. Several devices are automatically added in terms of power or, in the case of battery storage, the average state of charge is calculated.

**For example**:

```yaml
site:
  - title: Home # display name for UI
    meters:
      grid: mygridmeter # grid meter reference
      pv: # (pvs = deprecated)
        - mypv1 # first pv meter reference
        - mypv9 # second pv meter reference
      battery: # (batteries = deprecated)
        - mybat5 # battery meter reference
      aux:
        - myaux1
    residualPower: 100
    bufferSoc: 80
    prioritySoc: 66
```

---

## Required Parameters

### `title`

The description of the charging point is also displayed in the UI.

**For example**:

```yaml
title: Home
```

---

### `meters`

Defines which configured meter (current measurement devices) are to be used as what type of measurement point. This logically links the device definition with its intended use. A initially universal meter is thus assigned a purpose based on its location within the home installation.

:::note
At least the configuration of one `grid` or at least one `pv` element is necessary!
evcc cannot be used without at least one of these entries!
:::

**For example**:

```yaml
site:
  meters:
    grid: mygridmeter # grid meter reference
    pv: mypv1 # pv meter reference
    battery: mybat2 # battery meter reference
    aux: myaux1
```

---

## Optional Parameters

### `meters.grid`

Defines the [`meter`](meters) (current measurement device) that provides the measurements of the grid connection point.

**Possible values**: Value of a `name` parameter in the [`meters`](#meters) configuration.

**For example**:

```yaml
grid: mygridmeter # grid meter reference
```

---

### `meters.pv`

Defines [`meter`](meters) (current measurement devices), from which evcc fetches PV generation measurements.
Multiple devices can be specified. Power data is automatically added.

**Possible values**: A single value or a list of values of a name parameter in the [`meters`](#meters) configuration. The list version can also be used with single values.

**For example**:

```yaml
pv: myonlypv # singele pv meter reference
```

or

```yaml
pv: # (pvs = deprecated)
  - myoldpv # first pv meter reference
  - mynewestpv # second pv meter reference
```

---

### `meters.battery`

Defines the [`meter`](meters) (current measurement devices) that provide measurement data from the battery(ies).
Multiple devices can be specified. Power data is automatically added, and an average is calculated from the battery state of charge.

**Possible values**: A single value or a list of values of a `name` parameter in the [`meters`](#meters) configuration. The list version can also be used with single values.

**For example**:

```yaml
battery: myonlybat # single battery meter reference
```

or

```yaml
battery: # (batteries = deprecated)
  - mysmallbat # first battery meter reference
  - myhugebat # second battery meter reference
```

### `meters.aux`

Defines the meters (current measurement devices) that provide measurement data from external devices that have their own surplus control but are not directly controlled by evcc. Multiple devices can be specified. Power data is automatically added.

In evcc, this power contributes to the calculation of the available surplus power for vehicle charging. It is assumed that devices measured using auxiliary meters will autonomously and promptly reduce or completely interrupt their power consumption when the measured auxiliary power is allocated to vehicle charging by evcc.

Positive value: additional available surplus power (allocated to vehicle charging)

Negative value: insufficient surplus power (not available for vehicle charging)

Examples:

    An immersion heater for hot water production that is autonomously regulated based on PV surplus at the grid connection point. When the power consumption of this immersion heater is measured and configured as an aux meter, the entire surplus power (power of the immersion heater plus any remaining grid feed-in) is always preferentially allocated to vehicle charging. If the vehicle charging accesses it, the autonomous regulation of the immersion heater ensures that its power is reduced accordingly.

**Possible values**: A single value or a list of values of a `name` parameter in the [`meters`](#meters) configuration. The list version can also be used with single values.

**Example**:

```yaml
aux: myaux # single aux meter reference
```

or

yaml

```yaml
aux:
  - myaux1 # first aux meter reference
  - myaux2 # second aux meter reference
```

### `prioritySoc` 
#### (in Battery UI, boundary between Home and Vehicle)

Charging the house battery is prioritized over vehicle charging below the specified state of charge (SoC) (%) value. If there is more generation capacity available below this value than the battery can absorb, this surplus can still be used for vehicle charging, but as a lower priority. When the house battery is charged above this value, the battery charging power is considered as available surplus power for vehicle charging. Thus, vehicle charging takes priority in using the surplus power when it is above this SoC value.
It is disabled (equivalent to 0%) if no value is provided.

:::note
`prioritySoc` must have a smaller value than `bufferSoc`.
:::

**For example**:

```yaml
prioritySoc: 50 # House battery has priority for charging up to 50% SoC
```

### `bufferSoc` 
#### (in Battery UI, boundary between Vehicle and Battery-assisted Charging)

Allows the discharging of a house battery above the specified state of charge (SoC) (%) value when there is insufficient solar surplus (below the minimum charging power). This helps balance fluctuations in generation or consumption primarily using the house battery. If the discharge power of the house battery is not enough to provide the minimum charging power for the vehicle, the remainder will be sourced from the grid.

In `PV` mode, a charging process is automatically initiated when enough solar surplus is available.

It is disabled (equivalent to >100%) if no value is provided.

:::note
`bufferSoc` must have a larger value than `prioritySoc`.
:::

**For example**:

```yaml
bufferSoc: 80 # House battery is used as a buffer above 80% SoC
```

### `bufferStartSoc` 
#### (in Battery UI, boundary in Battery-assisted Charging Range)

In `PV` mode, allows the start of a charging process above the specified state of charge (SoC) (%) value, even if there is insufficient solar surplus.

If the discharge power of the house battery is not enough to provide the minimum charging power for the vehicle, the remainder will be sourced from the grid.

It is disabled (equivalent to 0%) if no value is provided.

:::note
`bufferStartSoc` must have a larger value than `bufferSoc`.
:::

**For example**:

```yaml
bufferStartSoc: 90 # Charging process starts when the house battery reaches 90% SoC
```

### `residualPower`

Sets the target operating point of the surplus regulation at the grid connection (grid meter). The default value is 0 W.
Negative values shift the target value towards grid feed-in, while positive values shift it towards grid consumption.
Ultimately, this value sets the "idle state" of the control loop that needs to be adjusted by the controller.

Especially in combination with other independent surplus regulation systems like a battery storage, this value must be adjusted to achieve a defined system behaviour with clear priorities.

If a certain proportion of grid consumption should remain or be allowed in PV mode, a negative value corresponding to the maximum proportion of grid consumption can be configured.

#### `grid` `meter` present

- Positive value: Remaining grid feed-in power
- Negative value: Remaining grid consumption power

#### Only `pv` `meter` present

- Positive value: Typical household consumption, used to estimate PV surplus.
- Negative value: The specified power is added to PV power and increases available charging power (Attention: grid consumption)

:::info
When a battery storage is present, it is strongly recommended to enter a small value, e.g. between 100 to 300 W here, allowing battery charging according to configured priorities (see `prioritySoc`). Otherwise, the independent regulation of the storage will not see usable surplus. Likewise, this prevents temporary grid consumption even without a battery during rapid generation and load changes.
:::

**Example "Battery Storage"**:

```yaml
residualPower: 100
```

**For example "Grid Consumption Proportion"**:

Charging should start in PV mode with at least 6A (single-phase) even with only 50% PV surplus (rest from grid).
Minimum charging power: 1 phase _ 6A _ 230V = 1380 W, 50% of it: 690 W

```yaml
residualPower: -690
```

### `smartCostLimit` (formerly `cheap`)

This parameter can set a price or gCO2-equivalent limit. In PV mode, charging starts when this limit is undercut.

Example price limit (for variable tariffs):

```yaml
smartCostLimit: 0.20 # 20 cents (or pence)
```

For example gCO2e limit (when using e.g. green energy index):

```yaml
smartCostLimit: 550 # 550 gCO2 equivalent
```

### `maxGridSupplyWhileBatteryCharging`

This parameter is only helpful for hybrid inverter systems where the DC power generated in combination with a directly connected storage system can be greater than the AC output power of the inverter. This can result in grid consumption during vehicle charging, even when the battery is being charged simultaneously.

Scenario:
Maximum 10 kW AC output power of the hybrid inverter. Current PV generation is 15 kW, with 5 kW going into the directly connected battery since the AC path of the hybrid inverter is fully utilized at 10 kW.

Normally, the current battery charging power is considered as additional available vehicle charging power (once `prioritySoc` is reached). In the above example, grid consumption could occur equal to the current battery charging power since the inverter cannot provide this power to the grid (and thus the vehicle). The fully utilized AC path of the hybrid inverter creates an unexpected bottleneck for the standard regulation.

With this parameter, a threshold value for grid consumption can be set. If this threshold is reached, the battery charging power is not considered as available vehicle charging power in this scenario. This way, the maximum surplus charging power remains limited to the maximum AC output power of the inverter, plus this value.

A recommended value is at least 50 Watts. Depending on the responsiveness of the control systems involved, it might need to be higher.

```yaml
maxGridSupplyWhileBatteryCharging: 50
```
