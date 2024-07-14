---
layout: posts
title: BdNMC Code review_test_post
date: 2024-07-14 06:33:53
categories:
  - B_N__SGDD
tags: 
comments: false
---

![Github_Logo](/attachment/img/4ac0ff20e4064e.png)




# main.cpp



## Line 57-63, time_delay_fraction

```c++
double t_delay_fraction(double tcut, double pos, double speed) {  
    double tdelay = pos / speed / speed_of_light - pos / speed_of_light;  
    if (tdelay > tcut)  
        return 1;  
    else  
        return 1 - (tcut - tdelay) / tcut;  
}
```

## Line 65-166, parameter card error check and read the pre-simulation setting.

### Line 65-124, parameter card error check.
### Line 127-130, Random seed init

```c++
if (par->Seed() >= 0)  
    Random(par->Seed());  
else  
    Random();
```

- random seed 가 제공되면 그걸로 initialize,
- 없는 경우는 그냥 랜덤하게 하나 지정

### Line 134, [[1609.01770.pdf#page=22&selection=55,0,55,10|Timing cut]]

- time delay 와 관련된 부분

### Line 136-166, parameter card read

#### Line 136-143, output & summary file location and name
#### Line 146-151, signal channel, signal particle
#### Line 153-164, output mode define
```c++
string outmode;  
std::unique_ptr <std::ofstream> comprehensive_out;
//if((outmode=par->Output_Mode())=="summary")  
if ((outmode = par->Output_Mode()) == "comprehensive" || outmode == "dm_detector_distribution") {  
    comprehensive_out = std::unique_ptr<std::ofstream>(new std::ofstream(output_filename));  
    if (!comprehensive_out->is_open()) {  
        cout << "Unable to open output file: " << output_filename << endl;  
        parstream.close();  
        return -1;  
    }}

```

## Line 168-275, Run parameter, Detector setup, Production mode

- 여기까진 pMax  가 나오지 **않았다.**

