
# CSE 566S Homework 1
## Christopher Ogle
___

## Problem 1

___


<ul>
    <li><h3>Gprof</h3></li>
    <ul>
        <li>
            <p>
                Of the three Performance Profilers analyzed in the assignment this profiler 
                seemed to be the most documented and widely used. As a result getting on of 
                the advantages of this profiler is that it is slightly easier to get started 
                with. This is a plus if you are simply trying to quickly profile something as
                there is ample documentation and setup is not extremely arduous. 
            </p>
        </li>
        <li>
            <p>
                While simple to use Gprof does have its limitations. One of the most glaring is
                its inability to measure the performance of multi-threaded code. When created
                multi-core systems were a luxury and thus optimizing over such machines was not 
                needed; however, in today's world where multi-core machines are the norm 
                multi-threaded code is no longer a rarity.
            </p>
        </li>
        <li>
            <p>
                Gprof measures the performance of applications by using a method called
                sampling. When sampling the code the it only measures the time when the code
                itself is running. Thus if the process is switched out by the CPU it does NOT
                measure this time. While this might seem like a benefit it can actually hide
                problem areas in our code. Our code might have a bottle neck in an external
                library or maybe even in an IO operation. However this problem would never be
                realized and while you might spend hours and hours improving your code you would
                never realize the problem isn't the code but rather from an external device.
                In addition because we are sampling the code there is the possibility that
                our code runs so fast that during the sampling time a function never gets
                registered as being called, and therefore we might have slightly skewed
                results.
            </p>
        </li>
        <li>
            <p>
               The final issue with Gprof is that optimized code can reek havoc on the
               profiling. When compiling with the -O3 flag the compiler will inline
               functions. Thus from the view of the profiler functions have disappeared and
               no longer will you be able to measure their performance.
            </p>
        </li>
    </ul>
    <li><h3>Oprofile</h3></li>
    <ul>
        <li>
            <p>
               Like Gprof, Oprofile is a statistical sampling profiler. Oprofile doesn't
               just sample only when your process is running, instead it continuously
               samples even when your code has stopped running due to a context switch. This
               solves one of the problems mention above. The source of a bottleneck might
               not be from the code itself, instead it might be from an external library or
               poor IO/cache performance. This continuous polling allows problems such as
               these to be recognized and dealt with. 
            </p>
        </li>
        <li>
            <p>
               One issue with Oprofile is that the user must run this command as root. While
               this is not an issue for somebody using their personal machine. If they are
               using a machine where they do not have such privileges then using this tool
               may prove to be impossible. Since we are performing a continuous poll and are
               root it is now possible to view all processes on the CPU, and thus we now
               have the ability to monitor multi-threaded code. While this tool has the
               caveat that it must be ran in sudo, it does make itself much more useful by
               supporting the profiling of multi-threaded code.
            </p>
        </li>
        <li>
            <p>
               Unlike Gprof, Oprofile does NOT require recompiling each time you wish to
               run a performance test. This means that if I wish to rerun a test I do not
               need to recompile the source code in order to generate a new test. This is a
               benefit in that source code can sometimes take a pretty long time to build.
               At a company I previously interned at I waited on average about 10 minutes 
               for the companies source code to completely build. Having to do that every 
               time one wants to measure performance is a costly waste of time. Luckily 
               Oprofile has no such requirements and does not need to insert itself directly 
               into the code. 
            </p>
        </li>
    </ul>
    <li><h3>Perf</h3></li>
    <ul>
    <li>
        <p>
               One of the immediate things that I noticed about this profiler is that there
               is a lot less information on the web regarding this profiler compared to the
               previous two. The Linux man pages contains a large document about
               PERF_EVENT_OPEN(2) which is useful, but does require sometime to read and
               understand as it aimed towards an advanced level programmer.
        </p>
    </li>
    <li>
        <p>
               Perf is similar to Oprofile in the sense that it can measure multi-threaded
               code, does not need to be compiled with the code in order to run the test and
               is a statistical sampling profiler.
        </p>
    </li>
    </ul>
</ul>

___

## Problems 2 and 4

<center><h3>Data Collection Methodology</h3></center>
<p>
An important thing to discuss is how data was collected for the following
trials. Every command was ran from the command line, no script was used to gather
data. In addition in between each run a tab the Firefox browser was ran for a
minute where I visited various sites, in order hopefully clear the cache so
that each test would have a similar random starting state. Each level of 
optimization was ran three times. The output of each test has been stored and
is located in the Results folder. The machine used was the campus linux machine.
</p>

___
<center><h3>Gprof Timing Results and Analysis</h3></center>


```python
import plotly.plotly as py
import plotly.graph_objs as go

#Times in seconds, pulled from Results folder.
y0 = [1.18, 1.18, 1.19]
y1 = [1.18, 1.18, 1.23]
y2 = [1.19, 1.17, 1.22]

#Each Traces represents a level of optimization
trace0 = go.Box(
    y=y0,
    name = 'No Optimization'
)
trace1 = go.Box(
    y=y1,
    name = 'Level 1 Optimization'
)
trace2 = go.Box(
    y=y2,
    name = 'Level 2 Optimization'
)
data = [trace0, trace1, trace2]

layout=go.Layout(height=750,
                 title="Gprof: Time to run vs Optimization", 
                 xaxis={'title':'Optimization Level'}, 
                 yaxis={'title':'Time (sec)'})

figure=go.Figure(data=data,layout=layout)
py.iplot(figure, filename='gprof_graph')
```




<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="https://plot.ly/~cogle/10.embed" height="750px" width="100%"></iframe>



<center><h3>Gprof Data Analysis</h3></center>

<p>
The results above for Gprof surprised me a bit. One of the first things that
is apparent from this graph is that I was unable to collect data past Level 2
Optimization. I was not 100 percent sure as to why this happened but my
hypothesis is was follows; -O3 optimization turns on 
<code>-finline-functions</code>, which is inhibiting Gprof 
from sticking in its timers. In order to test this idea I 
copied the original code and changed <code>vm_multiply()</code> to be and 
inline function and reran the gprof test with no optimization. 
</p>

<img src="https://raw.githubusercontent.com/cogle/CSE566_Homework_1/master/Results/gprof/Hypothesis/Hypothesis_snip.PNG?token=AE67yO2TfbhATypt7fkJZhxt01qCSIkdks5XetjnwA%3D%3D"></img>

<p>
Sure enough having inlined our function has caused its calls to disappear, 
yet the time to complete the program is on par with previously ran tests runs. 
Therefore, I believe that Optimization Level 3's aggressive optimization tactics
make it so that Gprof's timing hooks are no longer registered due to the 
<code>-finline-functions</code> optimization. 
</p>

___

<center><h3>Operf Timing Results and Analysis</h3></center>


```python
import plotly.plotly as py
import plotly.graph_objs as go

#CPU events registered, pulled from results folder
y0 = [15923, 15794, 15692]
y1 = [16120, 16428, 16083]
y2 = [15723, 16101, 15694]
y3 = [15798, 15680, 17318]
y4 = [9217, 8964, 8785]

#Each Traces represents a level of optimization
trace0 = go.Box(
    y=y0,
    name = 'No Optimization'
)
trace1 = go.Box(
    y=y1,
    name = 'Level 1 Optimization'
)
trace2 = go.Box(
    y=y2,
    name = 'Level 2 Optimization'
)
trace3 = go.Box(
    y=y3,
    name = 'Level 3 Optimization'
)
trace4 = go.Box(
    y=y4,
    name = 'Level Fast Optimization'
)
data = [trace0, trace1, trace2, trace3,trace4]

layout=go.Layout(height=1000,
                 title="Operf: Number of CPU Events vs Optimization Level", 
                 xaxis={'title':'Optimization Level'}, 
                 yaxis={'dtick': 500,
                        'title':'Number of CPU Events'})

figure=go.Figure(data=data,layout=layout)
py.iplot(figure, filename='operf_graph')
```




<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="https://plot.ly/~cogle/12.embed" height="1000px" width="100%"></iframe>



<center><h3>Operf Data Analysis</h3></center>

<p>
Using an interrupt polling system allowed us to collect far more data, than
inserting timing hooks which is the method that Gprof relies on. One of the
trends that is really hard to not notice is that up until performing Fast
Optimization each function call generated about the same number of CPU events.
I had expected to see a linear trend downward in the number of events because I
figured that as the optimizer optimizes that it would get rid of extraneous 
code. After looking at what exactly each level of optimization does I think that
while one might optimize the code it has to be in compliance with C strict
standards. However, compiling with Fast Optimization disregards this the 
compliance mode and optimizes for pure speed.    
</p>

___

<center><h3>Perf Timing Results and Analysis</h3></center>


```python
import plotly.plotly as py
import plotly.graph_objs as go

#CPU events registered, pulled from results folder
y0 = [1.217287294, 1.181955770, 1.183942236]
y1 = [1.227768100, 1.198169738, 1.211748535]
y2 = [1.196514761, 1.202492659, 1.219955811]
y3 = [1.189910993, 1.229740070, 1.303206873]
y4 = [0.701674410, 0.708538378, 0.537496915]

#Each Traces represents a level of optimization
trace0 = go.Box(
    y=y0,
    name = 'No Optimization'
)
trace1 = go.Box(
    y=y1,
    name = 'Level 1 Optimization'
)
trace2 = go.Box(
    y=y2,
    name = 'Level 2 Optimization'
)
trace3 = go.Box(
    y=y3,
    name = 'Level 3 Optimization'
)
trace4 = go.Box(
    y=y4,
    name = 'Level Fast Optimization'
)
data = [trace0, trace1, trace2, trace3,trace4]

layout=go.Layout(height=750,
                 title="Perf: Time to run vs Optimization", 
                 xaxis={'title':'Optimization Level'}, 
                 yaxis={'title':'Time (sec)'})

figure=go.Figure(data=data,layout=layout)
py.iplot(figure, filename='perf_graph')
```




<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="https://plot.ly/~cogle/18.embed" height="750px" width="100%"></iframe>



<center><h3>Perf Data Analysis</h3></center>

<p>
Similar to the results from the Operf we see that we never achieve a
significant drop in running time until running our code in Fast Optimization
mode. This profiler, like Operf, uses an interrupt polling system in order to
obtain its results, and again we see that unlike the problems we had with Gprof
we were able to collect data for every level of tested optimization. 
</p>

___

<center><h3>Problem 2 Answer</h3></center>
<p>
The above graphs highlight one of the problems with Gprof's manual insertion of
timing hooks. The insertion of timing hooks is highly dependent on the manner in
which the code is structured, altering this structure altering this structure
renders Gprof clueless as to what is going on. Thus as we increased the
optimization levels and more aggressive code restructuring occurred Gprof was
unable to measure any events. On the contrary with the interrupt polling that
Operf and Perf employ, we are able to monitor the function the entire time
regardless of the Optimization level. However, as we increase the Optimization
due to what I assume is the inlining of functions we lose granularity with
respect to which functions are being called as functions become inlined
</p>

<center><h3>Problem 4 Answer</h3></center>
<p>
One of the more startling results to come out of the above graphs is that the
Level of Optimization does not appear to decrease the amount of time or cycles
that the program takes, unless we compile with the Fast option. This result is
particularly surprising as I would have anticipated that there would have been
a slight decrease, but on the contrary it appears going from no optimization at
all to Level 3 Optimization actually seems to increase the time that it takes or
remain about the same. I will admit that my sampling is rather low and that in
order to make a much more definitive conclusion I would need to perform a much
more extensive sampling of each profiler. As for why this occurs I can not
really say. One reason that compiling with Optimization Level Fast is so much
faster is that it disregards the strict compliance standards. This however does
not explain why there is not a decreasing trend with respect to time/cycles over
optimization level.  
</p>

___

## Problem 5


```python
import plotly.plotly as py
import plotly.graph_objs as go
from plotly.tools import FigureFactory as FF 

# Create random data with numpy
import numpy as np

x = ['No Optimization', 'Level 1 Optimization', 
     'Level 2 Optimization', 'Level 3 Optimization',
     'Level Fast Optimization']

reg_no_opt = [15923,15794,15692,16197,16335]
rev_no_opt = [15771,16647,16523,16636,16117]

reg_opt_1 = [16120,16428,16083,16246,16456]
rev_opt_1 = [16617,15634,15717,16690,15818]

reg_opt_2 = [15723,16101,15694,16102,16235]
rev_opt_2 = [16413,15824,15831,15949,16295]

reg_opt_3 = [15798,15680,17318,18178,16442]
rev_opt_3 = [16412,15914,15527,15898,16140]

reg_opt_fast = [9217,8964,8785,9185,8846]
rev_opt_fast = [9272,9162,9114,9271,8846]

y0 = [np.average(reg_no_opt),
      np.average(reg_opt_1),
      np.average(reg_opt_2),
      np.average(reg_opt_3),
      np.average(reg_opt_fast)]

y1 = [np.average(rev_no_opt),
      np.average(rev_opt_1),
      np.average(rev_opt_2),
      np.average(rev_opt_3),
      np.average(rev_opt_fast)]

# Create traces
trace0 = go.Scatter(
    x = x,
    y = y0,
    mode = 'lines+markers',
    name = 'r = vM (Default)',
    error_y=dict(
        type='data',
        array=[np.std(reg_no_opt), 
               np.std(reg_opt_1),
               np.std(reg_opt_2),
               np.std(reg_opt_3),
               np.std(reg_opt_fast)],
        visible=True
    )
)
trace1 = go.Scatter(
    x = x,
    y = y1,
    mode = 'lines+markers',
    name = 'r = Mv (Reversed)',
    error_y=dict(
    type='data',
        array=[np.std(rev_no_opt), 
               np.std(rev_opt_1),
               np.std(rev_opt_2),
               np.std(rev_opt_3),
               np.std(rev_opt_fast)],
        visible=True
    )
)
data = [trace0, trace1]
layout=go.Layout(height=1000,
                 title="Operf: Number of CPU Events vs Optimization Level", 
                 xaxis={'title':'Optimization Level'}, 
                 yaxis={'dtick': 500,
                        'title':'Number of CPU Events'})

figure=go.Figure(data=data,layout=layout)
py.iplot(figure, filename='reverse-comparision')
```




<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="https://plot.ly/~cogle/14.embed" height="1000px" width="100%"></iframe>




```python
import plotly.plotly as py
from plotly.tools import FigureFactory as FF 

reg_no_opt = [15923,15794,15692,16197,16335]
rev_no_opt = [15771,16647,16523,16636,16117]

reg_opt_1 = [16120,16428,16083,16246,16456]
rev_opt_1 = [16617,15634,15717,16690,15818]

reg_opt_2 = [15723,16101,15694,16102,16235]
rev_opt_2 = [16413,15824,15831,15949,16295]

reg_opt_3 = [15798,15680,17318,18178,16442]
rev_opt_3 = [16412,15914,15527,15898,16140]

reg_opt_fast = [9217,8964,8785,9185,8846]
rev_opt_fast = [9272,9162,9114,9271,8846]



data_matrix = [['Run + Optimization Level', 'Average', 'Standard Deviation'],
               ['Default No Opt', np.average(reg_no_opt), np.std(reg_no_opt)],
               ['Reverse No Opt', np.average(rev_no_opt), np.std(rev_no_opt)],
               ['Default Opt Level 1', np.average(reg_opt_1), np.std(reg_opt_1)  ],
               ['Reverse Opt Level 1', np.average(rev_opt_1), np.std(rev_opt_1)  ],
               ['Default Opt Level 2', np.average(reg_opt_2), np.std(reg_opt_2)  ],
               ['Reverse Opt Level 2', np.average(rev_opt_2), np.std(rev_opt_2)  ],
               ['Default Opt Level 3', np.average(reg_opt_3), np.std(reg_opt_3)  ],
               ['Reverse Opt Level 3', np.average(rev_opt_3), np.std(rev_opt_3)  ],
               ['Default Opt Fast', np.average(reg_opt_fast), np.std(reg_opt_fast)],
               ['Reverse Opt Fast', np.average(rev_opt_fast), np.std(rev_opt_fast)]]

table = FF.create_table(data_matrix)
py.iplot(table, filename='results_table')
```




<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="https://plot.ly/~cogle/16.embed" height="380px" width="100%"></iframe>



<center><h3>Data Analysis and Conclusion</h3></center>
<p>
Above is the data collected for Problem 5. In this problem I measured  the number 
of CPU events for a given optimization level 5 times. From the graph above it is 
clear that there is no definitive answer as to whether or not switching around 
the order of multiplication speeds up or slows down the code. The graph clearly 
shows the <b>Default</b> and <b>Reversed</b> lines crossing throughout the 
experiment with neither line remaining above or below the other line. In addition 
for every optimization level their exist regions of overlap when both of line's 
standard deviations are taken into consideration. This points to a an experiment 
that yielded similar values despite the order of multiplication. 
</p>

___

## Problem 3

<p>
For this problem we are asked to look at the organization of the cache. One way
in which we can do this is by analyzing the Perf outputs. Perf contains a large
number of events that a user can track, amongst these trackable events are
cache-related events. This allows us the ability to analyze how the cache is
being used by our program. Somethings I noticed was that our program relies
very heavily on the L1 cache. The number of loads and evictions from this
particular cache far outnumber the other cache. One thing that this points to
is its importance. Now one reason that the number of the cache loads might be
so large is that typically the size of this cache is much smaller. This is due
to cost associated with this cache. Thus I would anticipate a larger number of
loads, as is seen. The other cache received less load request but it is likely
that the amount of data stored in this particular level is far greater than that
of the L1. 
</p>
