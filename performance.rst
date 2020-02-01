Performance
===========

`Slides <https://drive.google.com/drive/u/1/folders/1Z9wM7Bkrc1vo4GDyDt8d5y4eQACwXc2M>`_

Computer Architect
------------------
**Computer Architect**: To design the levels of a comp sys to maximize performance/power/programmability within tech limits (cost, etc)

Must be aware of:
    - measures of cost, performance, energy/power
    - applications characteristics/benchmarks
    - how to design processor/system blocks

Processor design usually takes 2-5 years, look to the future
    - 3-4 years from design to fabrication

**Moore's Law**: the density of transistors on a cip double each year (but in reality, more like every 3 yr)
    - *but*, it does not necessarily mean doubling performance
    - doubling performance every year would be exponential
    - Moore's Law cannot continue in 2D forever

Not all trends necessarily hold

Limits
------
**Power**: Available from a home outlet - about 1kW

**Cooling**: around 300W max from air cooling - most practical is around 100W

- power density: how much power you can draw in a spike
- mobile devices' battery limits/heat
- reliability at high temps
- energy cost/environmental

Current Trends
--------------
See slides - freq, power (cooling), and single thread perf are tapering off

Power Consumption in Chips
--------------------------

**Dynamic Power Consumption**: Charging and discharging capacitors, depends on switching transistors and activity (~90%)

**Leakage Current / Static**: Leaking diodes/transistors - gets worse with smaller devices, lower Vdd, higher temps (~10%)

Power = :math:`C * Vdd^2 * F_{0 \to 1} + Vdd * I_{leakage}`

Energy v. Power
---------------

Energy = Average power * Execution time
    - J = W * sec
    - Power is limited by infrastructure
    - Energy is what the utilities charge for

Improve energy by reducing power consumption, or improving execution time

Other Metrics
-------------

- performance/watt (e.g. qps/W)
- energy * delay (EDP)
- energy * delay^2 (EDP2)
    - since :math:`F_{0 \to 1} \sim Vdd` where F = frequency, and delay = 1/F

Performance Metrics
-------------------

**Performance**: For a machine A running a program P,

.. math::

    Perf(A,P) = \frac{1}{Time(A,P)}

    Perf(A,P) > Perf(B,P) \to Time(A,P) < Time(B,P)

**Speedup**: improved execution time factor (also commonly expressed in percent)

.. math::

    speedup = \frac{time(old)}{time(new)}

.. note::
    100% speedup means twice as fast!

Latency
^^^^^^^
The elapsed time it takes to do X

Medical defn: The interval between simulation and response

Latency is usually measured in percentile, not average
    - e.g. avg latency vs 99th percentile latency

CPUs (at least in this class) operate using a constant-rate clock

The duration of a clock period is the **clock cycle**

**clock frequency**: cycles/sec

    - e.g. 250ps clock cycle = 4GHz
    - 1ns = 1GHz

Throughput
^^^^^^^^^^
AKA Bandwidth

- the amount of material, data, etc, that enters and goes through something
- the rate that a unit of work is completed or started by a mechanism
- how much work X is done in a given time

Execution/CPU Time
^^^^^^^^^^^^^^^^^^^^^
Execution Time = Cycles per Program * Clock Cycle Time

Iron Law
^^^^^^^^
AKA CPU performance equation

Architecture * Implementation * Realization

IC = Instruction Count

CPI = Cycles per Instruction

.. math::

    Time = IC * CPI * Period_{clk}

CPI
^^^

**Instruction CPI**: each instruction has its own fixed CPI
    - e.g. CPI_load = 7
    - CPI_add = 5

**Average CPI** (most common): depends on program

calculated as :math:`CPI = \sum_{i=1}^n \frac{IC_i}{IC} * CPI_i` for each i in instructions

Amdahl's Law
------------
AKA "Partial Speedup"

Performance gain limited by the fraction of improved execution - speeding up only certain instruction types infinitely doesn't lead to infinite speedup

.. math::

    max speedup = \frac{1}{1-f(e)}

    speedup = \frac{1}{(1-f(e)) + \frac{f(e)}{speedup(e)}}

Where f(e) = the fraction of the instructions that is sped up

Summarizing Performance
-----------------------

ex: see slides perf02, pg 16 for table

========== ========== ========== ==========
Program    Computer A Computer B Computer C
---------- ---------- ---------- ----------
Program P1          1         10         20
Program P2       1000        100         20
Program P3       1001        110         40
Arith Mean       ~667        ~73        ~26
Geom Mean        ~100        ~48        ~25
========== ========== ========== ==========

Means
^^^^^
**Arithmetic Mean**: sum/num

**Geometric Mean**: nth root of n products - used in most benchmarks

**Harmonic Mean**: :math:`\frac{n}{\sum \frac{1}{perf(i)}}`

Usually, papers use geometric means of speedups across multiple benchmarks (see perf02, pg 18)

The geometric mean is agnostic to the normalization

TLDR:
    - arith < geo < harmonic
    - geometric and harmonic must be normalized
    - geometric is base independent

Benchmarks
----------

- programs used to measure performance
    - supposed to be representative of typical workload
- benchmark suite: a collection of benchmarks
    - plus datasets, metrics, and rules for evaluation
    - plus a way to quantify performance in a single number
- each one have different instruction counts/mix of instructions

a commonly used benchmark is SPEC
- SPECint: single thread master
- SPECfp: scientific apps
- SPECpower: 11 runs at (0, 10..100%) load
- SPECmp: noncommunicating parallel processes