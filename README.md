// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © tawinfish

//@version=4



strategy("[BOTBTC BY Tawin ]", overlay = true, pyramiding=1,initial_capital = 10000, default_qty_type= strategy.percent_of_equity, default_qty_value = 100, calc_on_order_fills=false, slippage=0,commission_type=strategy.commission.percent,commission_value=0.03)
ADX_options         =                   input("MASANAKAMURA",                   title="  ADX Option",                                       options = ["CLASSIC", "MASANAKAMURA"],                                          group = "ADX")
ADX_len             =                   input(4,                               title="  ADX Lenght",                                       type = input.integer, minval = 1,                                               group = "ADX")
th                  =                   input(4,                             title="  ADX Treshold",                                    type = input.float, minval = 0, step = 0.5,                                     group = "ADX")
calcADX(_len) =>
    up              =                                                                                                                       change(high)
	down            =                                                                                                                      -change(low)
	plusDM          =                                                                                                                       na(up)   ? na : (up > down and up > 0   ? up   : 0)
    minusDM         =                                                                                                                       na(down) ? na : (down > up and down > 0 ? down : 0)
	truerange       =                                                                                                                       rma(tr, _len)
	_plus           =                                                                                                                       fixnan(100 * rma(plusDM, _len)  / truerange)
	_minus          =                                                                                                                       fixnan(100 * rma(minusDM, _len) / truerange)
	sum             =                                                                                                                       _plus + _minus
	_adx            =                                                                                                                       100 * rma(abs(_plus - _minus) / (sum == 0 ? 1 : sum), _len)
    [_plus,_minus,_adx]
calcADX_Masanakamura(_len) =>
    SmoothedTrueRange                   =                                                                                                   0.0
    SmoothedDirectionalMovementPlus     =                                                                                                   0.0
    SmoothedDirectionalMovementMinus    =                                                                                                   0.0
    TrueRange                           =                                                                                                   max(max(high - low, abs(high - nz(close[1]))), abs(low - nz(close[1])))
    DirectionalMovementPlus             =                                                                                                   high - nz(high[1]) > nz(low[1]) - low ? max(high - nz(high[1]), 0) : 0
    DirectionalMovementMinus            =                                                                                                   nz(low[1]) - low > high - nz(high[1]) ? max(nz(low[1]) - low, 0)   : 0
    SmoothedTrueRange                   :=                                                                                                  nz(SmoothedTrueRange[1]) - (nz(SmoothedTrueRange[1]) /_len) + TrueRange
    SmoothedDirectionalMovementPlus     :=                                                                                                  nz(SmoothedDirectionalMovementPlus[1])  - (nz(SmoothedDirectionalMovementPlus[1])  / _len) + DirectionalMovementPlus
    SmoothedDirectionalMovementMinus    :=                                                                                                  nz(SmoothedDirectionalMovementMinus[1]) - (nz(SmoothedDirectionalMovementMinus[1]) / _len) + DirectionalMovementMinus
    DIP                                 =                                                                                                   SmoothedDirectionalMovementPlus  / SmoothedTrueRange * 100
    DIM                                 =                                                                                                   SmoothedDirectionalMovementMinus / SmoothedTrueRange * 100
    DX                                  =                                                                                                   abs(DIP-DIM) / (DIP+DIM)*100
    adx                                 =                                                                                                   sma(DX, _len)
    [DIP,DIM,adx]
[DIPlusC,DIMinusC,ADXC] =                                                                                                                   calcADX(ADX_len) 
[DIPlusM,DIMinusM,ADXM] =                                                                                                                   calcADX_Masanakamura(ADX_len)

DIPlus                  =                                                                                                                   ADX_options == "CLASSIC" ? DIPlusC    : DIPlusM
DIMinus                 =                                                                                                                   ADX_options == "CLASSIC" ? DIMinusC   : DIMinusM
ADX                     =                                                                                                                   ADX_options == "CLASSIC" ? ADXC       : ADXM
L_adx                   =                                                       DIPlus > DIMinus and ADX > th
S_adx                   =                                                       DIPlus < DIMinus and ADX > th
left                =                   input(10,                                title="  Left",                                                                                                                               group="Support and Resistance")
right               =                   input(1,                                title="  Right",                                                                                                                              group="Support and Resistance")
hih                 =                                                                                                                       pivothigh(high, left, right)
lol                 =                                                                                                                       pivotlow (low , left, right)
top                 =                                                                                                                       valuewhen(hih, high[right], 0)
bot                 =                                                                                                                       valuewhen(lol, low [right], 0)
RS_Long_condt       =                                                           close > top
RS_Short_condt      =                                                           close < bot
L_cross             =                                                           crossover(close, top)
S_cross             =                                                           crossunder(close,bot)
length_                =                   input(2,                                title="  Lenght",                                                                                                                               group="ARA")
gamma               =                   input(2,                                title="  Gamma",                                                                                                                              group="ARA")
zl = false
ma = 0.
mad = 0.
ma4h = 0.
mad4h = 0.
src_ = zl ? close + change(close,length_/2) : close
ma := nz(mad[1],src_)
d = cum(abs(src_[length_] - ma))/ bar_index * gamma
mad := sma(sma(src_ > nz(mad[1],src_) + d ? src_ + d : src_ < nz(mad[1],src_) - d ? src_ - d : nz(mad[1],src_),length_),length_)
mad_up = mad > mad[1]
Length1 = 500
SMA1 = sma(close, Length1)
Long_MA =  (SMA1 < close )
Short_MA = (SMA1 > close )
madup = mad > mad[1] ? #009688  : #f06292
mad_f = mad/mad[1] > .999 and mad/mad[1] < 1.001
var bool longCond = na, var bool shortCond = na
var int CondIni_long = 0, var int CondIni_short = 0
var bool _Final_longCondition = na, var bool _Final_shortCondition = na
var float last_open_longCondition = na, var float last_open_shortCondition = na
var int last_longCondition = na, var int last_shortCondition = na
var int last_Final_longCondition = na, var int last_Final_shortCondition = na
var int nLongs = na, var int nShorts = na
L_1     =                                                                       Long_MA and L_adx and RS_Long_condt  and not mad_f 
S_1     =                                                                       Short_MA and S_adx and RS_Short_condt  and not mad_f 
longCond                :=                                                      L_1
shortCond               :=                                                      S_1
CondIni_long                := longCond[1]              ? 1 :                   shortCond[1] ? -1 :                             nz(CondIni_long[1]                                          )
CondIni_short               := longCond[1]              ? 1 :                   shortCond[1] ? -1 :                             nz(CondIni_short[1]                                         )
longCondition               = (longCond[1]              and                                                                     nz(CondIni_long[1])                 == -1                   )
shortCondition              = (shortCond[1]             and                                                                     nz(CondIni_short[1])                ==  1                   )
var float sum_long = 0.0, var float sum_short = 0.0
var float Position_Price = 0.0
var bool Final_long_BB = na, var bool Final_short_BB = na
var int last_long_BB = na, var int last_short_BB = na
last_open_longCondition     :=                      longCondition               or          Final_long_BB[1]            ? close[1]      : nz(last_open_longCondition[1]                     )
last_open_shortCondition    :=                      shortCondition              or          Final_short_BB[1]           ? close[1]      : nz(last_open_shortCondition[1]                    )
last_longCondition          :=                      longCondition               or          Final_long_BB[1]            ? time          : nz(last_longCondition[1]                          )
last_shortCondition         :=                      shortCondition              or          Final_short_BB[1]           ? time          : nz(last_shortCondition[1]                         )
in_longCondition            =                       last_longCondition          >           last_shortCondition
in_shortCondition           =                       last_shortCondition         >           last_longCondition
last_Final_longCondition    :=                      longCondition               ? time                                                  :    nz(last_Final_longCondition[1]                 )
last_Final_shortCondition   :=                      shortCondition              ? time                                                  :    nz(last_Final_shortCondition[1]                )
nLongs                      :=                      nz(nLongs[1]                                                                                                                            )
nShorts                     :=                      nz(nShorts[1]                                                                                                                           )
if longCondition            or                      Final_long_BB
    nLongs                  :=                      nLongs                      + 1
    nShorts                 := 0
    sum_long                :=                      nz(last_open_longCondition) +           nz(sum_long[1])
    sum_short               := 0.0
if shortCondition           or                      Final_short_BB
    nLongs                  := 0
    nShorts                 :=                      nShorts + 1
    sum_short               :=                      nz(last_open_shortCondition)+ nz(sum_short[1])
    sum_long                := 0.0
    
Position_Price              :=                      nz(Position_Price[1])

Position_Price              :=                      longCondition               or          Final_long_BB       ?       sum_long/nLongs         :       shortCondition      or      Final_short_BB      ?       sum_short/nShorts       :       na
var bool long_tp = na, var bool short_tp = na
var int last_long_tp = na, var int last_short_tp = na
var bool Final_Long_tp = na, var bool Final_Short_tp = na
var bool Final_Long_sl0 = na, var bool Final_Short_sl0 = na
var bool Final_Long_sl = na, var bool Final_Short_sl = na
var int last_long_sl = na, var int last_short_sl = na
tp_long0            =                   input(0.8,                              title="  % TP Long",                                        type = input.float,     minval = 0,     step = 0.1,                           group="Target Point") 
tp_short0           =                   input(0.8,                              title="  % TP Short",                                       type = input.float,     minval = 0,     step = 0.1,                           group="Target Point") 
tp_long             =       ((nLongs  > 1)              ?                       tp_long0  / nLongs              :           tp_long0)                       / 100
tp_short            =       ((nShorts > 1)              ?                       tp_short0 / nShorts             :           tp_short0)                      / 100
long_tp             := high                             >                       (fixnan(Position_Price)         *           (1 + tp_long))                  and                 in_longCondition
short_tp            := low                              <                       (fixnan(Position_Price)         *           (1 - tp_short))                 and                 in_shortCondition
last_long_tp        :=      long_tp                     ?                       time : nz(last_long_tp[1])
last_short_tp       :=      short_tp                    ?                       time : nz(last_short_tp[1])
Final_Long_tp       :=      (long_tp                    and                     last_longCondition              >           nz(last_long_tp[1])             and                 last_longCondition  > nz(last_long_sl[1]))
Final_Short_tp      :=      (short_tp                   and                     last_shortCondition             >           nz(last_short_tp[1])            and                 last_shortCondition > nz(last_short_sl[1]))
L_tp                 =      iff(Final_Long_tp,                                  fixnan(Position_Price)          *           (1 + tp_long)                   ,                   na) 
S_tp                 =      iff(Final_Short_tp,                                 fixnan(Position_Price)          *           (1 - tp_short)                  ,                   na) 
tplLevel            = (in_longCondition                 and 
                      (last_longCondition               >                       nz(last_long_tp[1]))            and 
                      (last_longCondition               >                       nz(last_long_sl[1]))            and not Final_Long_sl[1])                   ? 
                      (nLongs > 1)                      ? 
                      (fixnan(Position_Price)           *                       (1 + tp_long))                  :               (last_open_longCondition    *              (1 + tp_long)) : na
tpsLevel            = (in_shortCondition                and 
                      (last_shortCondition              >                       nz(last_short_tp[1]))           and 
                      (last_shortCondition              >                       nz(last_short_sl[1]))           and not Final_Short_sl[1])                  ? 
                      (nShorts > 1)                     ? 
                      (fixnan(Position_Price)           *                       (1 - tp_short))                 :               (last_open_shortCondition   *             (1 - tp_short)) : na
sl0                 =                   input(3.2,                              title="  % Stop loss",                                        type = input.float,     minval = 0,     step = 0.1,                             group="Stop Loss")
Risk                = sl0
Percent_Capital     = 99
sl                  =  in_longCondition  ?  min(sl0,(((Risk) * 100) / (Percent_Capital *  max(1, nLongs))))  : 
                       in_shortCondition ?  min(sl0,(((Risk) * 100) / (Percent_Capital *  max(1, nShorts)))) : sl0                      
Normal_long_sl      =               ((in_longCondition                and low                             <= ((1 - (sl / 100))    *               (fixnan(Position_Price)))))
Normal_short_sl     =               ((in_shortCondition               and high                            >= ((1 + (sl / 100))    *               (fixnan(Position_Price)))))  
last_long_sl        :=              Normal_long_sl      ? time : nz(last_long_sl[1])
last_short_sl       :=              Normal_short_sl     ? time : nz(last_short_sl[1])
Final_Long_sl       :=              Normal_long_sl      and last_longCondition              > nz(last_long_sl[1])               and last_longCondition  > nz(last_long_tp[1])  and not Final_Long_tp
Final_Short_sl      :=              Normal_short_sl     and last_shortCondition             > nz(last_short_sl[1])              and last_shortCondition > nz(last_short_tp[1]) and not Final_Short_tp
if Final_Long_tp                    or                                          Final_Long_sl
    CondIni_long    :=                                                          -1
    sum_long        :=                                                          0.0
    nLongs          :=                                                          na
    
if Final_Short_tp                   or                                          Final_Short_sl
    CondIni_short   :=                                                          1
    sum_short       :=                                                          0.0
    nShorts         :=                                                          na
ara_col = in_longCondition and not mad_f ? #009688 : in_shortCondition and not mad_f ? #f06292 : color.gray

Bar_color = L_adx ? #009688 : S_adx ? #f06292 : color.orange
barcolor                                                                        (color = Bar_color)
plot(L_tp,                                                  title = "TP_L",                 style = plot.style_cross,                                                                 color = color.fuchsia,                          linewidth = 7   )
plot(S_tp,                                                  title = "TP_S",                 style = plot.style_cross,                                                                 color = color.fuchsia,                          linewidth = 7   )
plot(mad, title="ARA",  color=ara_col, style = plot.style_cross, linewidth=4, transp=0)

plot((nLongs > 1) or (nShorts > 1) ? Position_Price : na, title = "Price", color = in_longCondition ? color.aqua : color.orange, linewidth = 2, style = plot.style_cross)
plot(tplLevel,                      title="Long TP ",               style = plot.style_cross,                                                                   color=color.fuchsia,                                                                                      linewidth = 1               )
plot(tpsLevel,                      title="Short TP ",              style = plot.style_cross,                                                                   color=color.fuchsia,                                                                                        linewidth = 1               )
plotshape(Final_Long_tp,            title="TP Long Signal",         style = shape.triangledown,                     location=location.abovebar,                         color=color.red,            size=size.tiny ,          text="TP",             textcolor=color.red,               transp = 0                  ) 
plotshape(Final_Short_tp,           title="TP Short Signal",        style = shape.triangleup,                     location=location.belowbar,                         color=color.green,          size=size.tiny ,            text="TP",           textcolor=color.green,             transp = 0                  ) 

plotshape(longCondition,            title="Long",                   style=shape.triangleup,                 location=location.belowbar,                         color=color.blue,           size=size.tiny ,                                                           transp = 0                  )
plotshape(shortCondition,           title="Short",                  style=shape.triangledown,               location=location.abovebar,                         color=color.red,            size=size.tiny ,                                                           transp = 0                  )

plotshape(Final_Long_sl,            title="SL Long",                style=shape.xcross,                     location=location.belowbar,                         color=color.fuchsia,        size=size.small ,       text ="SL",                                         transp = 0                  ) 
plotshape(Final_Short_sl,           title="SL Short",               style=shape.xcross,                     location=location.abovebar,                         color=color.fuchsia,        size=size.small ,       text ="SL",                                         transp = 0                  ) 
ACT_BT              =                   input(true,                             title="Backtest",                                           type = input.bool,                                                              group= "BACKTEST")
testStartYear       =                   input(1997,                             title="start year",                                         minval = 1997, maxval = 3000,                                                   group= "BACKTEST") 
testStartMonth      =                   input(06,                               title="start month",                                        minval = 1, maxval = 12,                                                        group= "BACKTEST")
testStartDay        =                   input(01,                               title="start day",                                          minval = 1, maxval = 31,                                                        group= "BACKTEST")
testPeriodStart     =                   timestamp(testStartYear,testStartMonth,testStartDay,0,0)
testStopYear        =                   input(3333,                             title="stop year",                                          minval=1980, maxval = 2222,                                                     group= "BACKTEST")
testStopMonth       =                   input(12,                               title="stop month",                                         minval=1, maxval=12,                                                            group= "BACKTEST")
testStopDay         =                   input(31,                               title="stop day",                                           minval=1, maxval=31,                                                            group= "BACKTEST")
testPeriodStop      =                   timestamp(testStopYear, testStopMonth, testStopDay, 0, 0)
testPeriod          =                   time >= testPeriodStart and time <= testPeriodStop ? true : false
if                                                                              L_1
    strategy.entry                                                              ("L", strategy.long , when = ACT_BT and testPeriod)
if                                                                              S_1
    strategy.entry                                                              ("S", strategy.short, when = ACT_BT and testPeriod)   

strategy.exit("TP_L", "L", profit = (abs((last_open_longCondition  * (1 + tp_long)) - last_open_longCondition) / syminfo.mintick), limit = nLongs >= 1 ? strategy.position_avg_price * (1 + tp_long) : na, loss = (abs((last_open_longCondition*(1-(sl/100)))-last_open_longCondition)/syminfo.mintick))
strategy.exit("TP_S", "S", profit = (abs((last_open_shortCondition * (1 - tp_short)) - last_open_shortCondition) / syminfo.mintick), limit = nShorts >= 1 ? strategy.position_avg_price*(1-(tp_short)) : na, loss     = (abs((last_open_shortCondition*(1+(sl/100)))-last_open_shortCondition)/syminfo.mintick))
  












