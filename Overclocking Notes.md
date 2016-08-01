
Hardware
---------

* [Intel Core i7 930 @ 2.8Ghz](http://ark.intel.com/products/41447/Intel-Core-i7-930-Processor-8M-Cache-2_80-GHz-4_80-GTs-Intel-QPI)
* [ASUS P6X58D Premium](https://www.asus.com/Motherboards/P6X58D_Premium/)
* [Coolermaster Hyper 212 Plus Heatsink](http://www.coolermaster.com/cooling/cpu-air-cooler/hyper-212-plus/) with [Arctic Silver](https://www.amazon.com/Arctic-Silver-AS5-3-5G-Thermal-Paste/dp/B000OGX5AM) thermal paste

Settings for 3.8Ghz
-------------------

Works very well and with good temps. ~45C resting and ~80C peaks at 100%.
4.0Ghz would've been nice but this will work for me very well.

    Ai Overclock Tuner.....................[Manual]
    CPU Ratio Setting......................[21.0]
    Intel(r) SpeedStep(tm) Tech............[Disabled]
    Xtreme Phase Full Power Mode...........[Enabled]
    BCLK Frequency.........................[181]
    PCIE Frequency.........................[100]
    DRAM Frequency.........................[DDR3-1451MHz]
    UCLK Frequency.........................[2903MHz]
    QPI Link Data Rate.....................[Auto]

    CPU Voltage Control....................[Manual]
    CPU Voltage............................[1.25]
    CPU PLL Voltage........................[1.80]
    QPI/DRAM Core Voltage..................[1.25000]
    IOH Voltage............................[1.14]
    IOH PCIE Voltage.......................[1.50]
    ICH Voltage............................[1.20]
    ICH PCIE Voltage.......................[1.50]
    DRAM Bus Voltage.......................[1.64]
    DRAM DATA REF Voltage on CHA...........[Auto]
    DRAM CTRL REF Voltage on CHA...........[Auto]
    DRAM DATA REF Voltage on CHB...........[Auto]
    DRAM CTRL REF Voltage on CHB...........[Auto]
    DRAM DATA REF Voltage on CHC...........[Auto]
    DRAM CTRL REF Voltage on CHC...........[Auto]

    Load-Line Calibration..................[Enabled]
    CPU Differential Amplitude.............[800mV]
    CPU Clock Skew.........................[Delay 300ps]
    CPU Spread Spectrum....................[Disabled]
    IOH Clock Skew.........................[Auto]
    PCIE Spread Spectrum...................[Disabled]

    C1E Support............................[Disabled]
    Hardware Prefetcher....................[Enabled]
    Adjacent Cache Line Prefetch...........[Enabled]
    Intel(r) Virtualization Tech...........[Disabled]
    CPU TM Function........................[Enabled]
    Execute Disable Bit....................[Enabled]
    Intel(r) HT Technology.................[Enabled]
    Active Processor Cores.................[All]
    A20M...................................[Disabled]
    Intel(r) SpeedStep(tm) Tech............[Disabled]
    Intel(r) C-STATE Tech..................[Disabled]

Experiments around 4.0Ghz
-------------------------

Same as 3.8Ghz except

    BCLK Frequency.........................[190]
    CPU Voltage............................[1.35]
    DRAM Frequency.........................[DDR3-1531MHz]
    UCLK Frequency.........................[3063MHz]

Not stable.

    BCLK Frequency.........................[191]
    CPU Voltage............................[1.3]

Runs quite hot with maxes of 95C and is not stable.

    BCLK Frequency.........................[190]
    CPU Voltage............................[1.28125]

Not stable. I don't care and I give up.

Resources
---------

* [CPU-Z](http://www.cpuid.com) for CPU information
* [Real Temp](https://www.techpowerup.com/downloads/2089/real-temp-3-70/mirrors) for to view temperature (v3.70)
* [LinX](https://github.com/sanekgusev/LinX-old/releases/tag/0.6.5) for 'quick' stability testing.
* [Prime95 for long-term stability-testing](http://www.mersenne.org/download/#download).
  This is to be used after LinX yields a good result (keep an eye on temp)
* [PerformanceTest](http://www.passmark.com/download/pt_download.htm) for a quick before and after.
* Forum threads, etc
    - http://www.overclock.net/t/662236/official-the-asus-p6x58d-premium-e-thread
    - http://www.overclock.net/t/798321/oc-i7-930-on-asus-p6x58d-e#post_10321869
    - http://www.overclockers.com/3-step-guide-overclock-core-i3-i5-i7/
