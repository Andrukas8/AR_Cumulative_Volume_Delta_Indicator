//+------------------------------------------------------------------+
//|                                    AR_Cumulative_Delta_Index.mq5 |
//|                                  Copyright 2022, MetaQuotes Ltd. |
//|                                             https://www.mql5.com |
//+------------------------------------------------------------------+
// Inspired by this article: https://www.mql5.com/en/articles/10

#include <MovingAverages.mqh>
#property copyright "Copyright 2022, MetaQuotes Ltd."
#property link      "https://www.mql5.com"
#property version   "1.00"

#property indicator_separate_window

#property indicator_buffers 14
#property indicator_plots 1

#property indicator_type1   DRAW_COLOR_HISTOGRAM
#property indicator_label1  "Cumulative Delta Volume"
#property indicator_color1  clrGreen,clrIndianRed
#property indicator_width1  5
#property indicator_level1  0.0
#property indicator_levelstyle STYLE_SOLID
#property indicator_levelcolor DimGray

input int CDIPeriod = 10; // Period
input int CDIRange = 100; // Range

double cumulative_volume_delta_Buffer[];
double upper_wick_Buffer[];
double lower_wick_Buffer[];
double spread_Buffer[];
double body_length_Buffer[];
double percent_upper_wick_Buffer[];
double percent_lower_wick_Buffer[];
double percent_body_length_Buffer[];
double buying_volume_Buffer[];
double selling_volume_Buffer[];
double cumulative_buying_volume_Buffer[];
double cumulative_selling_volume_Buffer[];
double HistogramColorBuffer[];

string text; // text for debugging

//+------------------------------------------------------------------+
//| Custom indicator initialization function                         |
//+------------------------------------------------------------------+
int OnInit()
  {
//--- indicator buffers mapping
   SetIndexBuffer(0,cumulative_volume_delta_Buffer,INDICATOR_DATA);
   SetIndexBuffer(1,HistogramColorBuffer,INDICATOR_COLOR_INDEX);
   SetIndexBuffer(2,upper_wick_Buffer,INDICATOR_CALCULATIONS);
   SetIndexBuffer(3,lower_wick_Buffer,INDICATOR_CALCULATIONS);
   SetIndexBuffer(4,spread_Buffer,INDICATOR_CALCULATIONS);
   SetIndexBuffer(5,body_length_Buffer,INDICATOR_CALCULATIONS);
   SetIndexBuffer(6,percent_upper_wick_Buffer,INDICATOR_CALCULATIONS);
   SetIndexBuffer(7,percent_lower_wick_Buffer,INDICATOR_CALCULATIONS);
   SetIndexBuffer(8,percent_body_length_Buffer,INDICATOR_CALCULATIONS);
   SetIndexBuffer(9,buying_volume_Buffer,INDICATOR_CALCULATIONS);
   SetIndexBuffer(10,selling_volume_Buffer,INDICATOR_CALCULATIONS);
   SetIndexBuffer(11,cumulative_buying_volume_Buffer,INDICATOR_CALCULATIONS);
   SetIndexBuffer(12,cumulative_selling_volume_Buffer,INDICATOR_CALCULATIONS);

   ArraySetAsSeries(cumulative_volume_delta_Buffer,false);
   ArraySetAsSeries(HistogramColorBuffer,false);
   ArraySetAsSeries(upper_wick_Buffer,false);
   ArraySetAsSeries(lower_wick_Buffer,false);
   ArraySetAsSeries(spread_Buffer,false);
   ArraySetAsSeries(body_length_Buffer,false);
   ArraySetAsSeries(percent_upper_wick_Buffer,false);
   ArraySetAsSeries(percent_lower_wick_Buffer,false);
   ArraySetAsSeries(percent_body_length_Buffer,false);
   ArraySetAsSeries(buying_volume_Buffer,false);
   ArraySetAsSeries(selling_volume_Buffer,false);
   ArraySetAsSeries(cumulative_buying_volume_Buffer,false);
   ArraySetAsSeries(cumulative_selling_volume_Buffer,false);

   PlotIndexSetInteger(0,PLOT_COLOR_INDEXES,2);
   PlotIndexSetInteger(0,PLOT_DRAW_BEGIN,CDIPeriod-1);

   string shortname;
   StringConcatenate(shortname,"CDI ( Averaging = ",CDIPeriod," Bars range = ",CDIRange," )");
   PlotIndexSetString(0,PLOT_LABEL,shortname);
   IndicatorSetString(INDICATOR_SHORTNAME,shortname);
   IndicatorSetInteger(INDICATOR_DIGITS,2);

//---
   return(INIT_SUCCEEDED);
  }
//+------------------------------------------------------------------+
//| Custom indicator iteration function                              |
//+------------------------------------------------------------------+
int OnCalculate(const int rates_total,
                const int prev_calculated,
                const datetime &time[],
                const double &open[],
                const double &high[],
                const double &low[],
                const double &close[],
                const long &tick_volume[],
                const long &volume[],
                const int &spread[])
  {
//---
   if(rates_total<CDIPeriod)
     {
      Print("Not enough rates...");
      return(0);
     }

//--- if it is the first call
   if(prev_calculated==0)
     {
      //--- set zero values to zero indexes
      cumulative_volume_delta_Buffer[0] = 0.0;
      upper_wick_Buffer[0] = 0.0;
      lower_wick_Buffer[0] = 0.0;
      spread_Buffer[0] = 0.0;
      body_length_Buffer[0] = 0.0;
      percent_upper_wick_Buffer[0] = 0.0;
      percent_lower_wick_Buffer[0] = 0.0;
      percent_body_length_Buffer[0] = 0.0;
      buying_volume_Buffer[0] = 0.0;
      selling_volume_Buffer[0] = 0.0;
      cumulative_buying_volume_Buffer[0] = 0.0;
      cumulative_selling_volume_Buffer[0] = 0.0;
      HistogramColorBuffer[0] = 0.0;
     }
//--- calculate values of mtm and |mtm|
   int start;
   if(prev_calculated==0)
      start=rates_total - CDIRange;
   else
      start=prev_calculated-1;

   for(int i=start; i<rates_total; i++)
     {
      //upper_wick_Buffer
      if(close[i]>open[i])
         upper_wick_Buffer[i] = NormalizeDouble(high[i]-close[i],_Digits);
      else
         upper_wick_Buffer[i] = NormalizeDouble(high[i]-open[i],_Digits);

      //lower_wick_Buffer
      if(close[i]>open[i])
         lower_wick_Buffer[i] = NormalizeDouble(open[i]-low[i],_Digits);
      else
         lower_wick_Buffer[i] = NormalizeDouble(close[i]-low[i],_Digits);

      // spread_Buffer
      spread_Buffer[i] = NormalizeDouble(high[i] - low[i],_Digits);

      // body_length_Buffer
      body_length_Buffer[i] = NormalizeDouble(spread_Buffer[i] - (upper_wick_Buffer[i] + lower_wick_Buffer[i]),5);

      //percent_upper_wick
      percent_upper_wick_Buffer[i] = NormalizeDouble(upper_wick_Buffer[i]/spread_Buffer[i],5);

      //percent_lower_wick = lower_wick/spread
      percent_lower_wick_Buffer[i] = NormalizeDouble(lower_wick_Buffer[i]/spread_Buffer[i],5);

      //percent_body_length = body_length/spread
      percent_body_length_Buffer[i] = NormalizeDouble(body_length_Buffer[i]/spread_Buffer[i],5);

      //buying_volume
      if(close[i]>open[i])
         buying_volume_Buffer[i] = NormalizeDouble((percent_body_length_Buffer[i] + (percent_upper_wick_Buffer[i] + percent_lower_wick_Buffer[i])/2)*tick_volume[i],5);
      else
         buying_volume_Buffer[i] = NormalizeDouble(((percent_upper_wick_Buffer[i] + percent_lower_wick_Buffer[i])/2)*tick_volume[i],5);

      //selling_volume
      if(close[i]<open[i])
         selling_volume_Buffer[i] = NormalizeDouble((percent_body_length_Buffer[i] + (percent_upper_wick_Buffer[i] + percent_lower_wick_Buffer[i])/2)*tick_volume[i],0);
      else
         selling_volume_Buffer[i] = NormalizeDouble(((percent_upper_wick_Buffer[i] + percent_lower_wick_Buffer[i])/2) * tick_volume[i],0);

     }

//cumulative_buying_volume   //cumulative_selling_volume
   ExponentialMAOnBuffer(rates_total,prev_calculated,0,CDIPeriod,buying_volume_Buffer,cumulative_buying_volume_Buffer);
   ExponentialMAOnBuffer(rates_total,prev_calculated,0,CDIPeriod,selling_volume_Buffer,cumulative_selling_volume_Buffer);

   if(prev_calculated==0)
      start=CDIPeriod-1; // set the starting index for input arrays
   else
      start=prev_calculated-1;    // set

   for(int i=start; i<rates_total; i++)
     {
      //cumulative_volume_delta_Buffer
      cumulative_volume_delta_Buffer[i] = NormalizeDouble(cumulative_buying_volume_Buffer[i] - cumulative_selling_volume_Buffer[i],5);

      // Histogram Coloring
      if(cumulative_volume_delta_Buffer[i] > 0)
         HistogramColorBuffer[i]=0;
      else
         HistogramColorBuffer[i]=1;

      //For debugging
      //      text="";
      //      text += DoubleToString(upper_wick_Buffer[i],4) + " upper_wick_Buffer\n";
      //      text += DoubleToString(lower_wick_Buffer[i],4) + " lower_wick_Buffer\n";
      //      text += DoubleToString(spread_Buffer[i],4) + " spread_Buffer\n";
      //      text += DoubleToString(body_length_Buffer[i],4) + " body_length_Buffer\n";
      //      text += DoubleToString(percent_upper_wick_Buffer[i],4) + " percent_upper_wick_Buffer\n";
      //      text += DoubleToString(percent_lower_wick_Buffer[i],4) + " percent_lower_wick_Buffer\n";
      //      text += DoubleToString(upper_wick_Buffer[i],4) + " upper_wick_Buffer\n";
      //      text += DoubleToString(percent_body_length_Buffer[i],4) + " percent_body_length_Buffer\n";
      //      text += DoubleToString(buying_volume_Buffer[i],4) + " buying_volume_Buffer\n";
      //      text += DoubleToString(percent_body_length_Buffer[i],4) + " percent_body_length_Buffer\n";
      //      text += DoubleToString(selling_volume_Buffer[i],4) + " selling_volume_Buffer\n";
      //      text += DoubleToString(cumulative_buying_volume_Buffer[i],4) + " cumulative_buying_volume_Buffer\n";
      //      text += DoubleToString(cumulative_selling_volume_Buffer[i],4) + " cumulative_selling_volume_Buffer\n";
      //      text += DoubleToString(cumulative_volume_delta_Buffer[i],4) + " cumulative_volume_delta_Buffer\n";
      //      text += DoubleToString(HistogramColorBuffer[i],4) + " HistogramColorBuffer\n";
      //      Comment(text);

     }

   return(rates_total);
  }


//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
