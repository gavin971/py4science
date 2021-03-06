===============================
 Matplotlib: Beyond the basics
===============================

Status and plan for today
=========================

By now you know the basics of:

- Numpy array creation and manipulation.
- Display of data in numpy arrays, sufficient for interactive exploratory work.

Hopefully after this week you will:

- Know how to polish those figures to the point where they can go to a journal.
- Understand matplotlib's internal model enough to:
  - know where to look for knobs to fine-tune
  - better understand the help and examples online
  - use it as a development platform for complex visualization


Matplotlib's main APIs: ``pyplot`` and object-oriented
======================================================

Matplotlib is a library that can be thought of as having two main ways of being
used:

- via ``pyplot`` calls, as a high-level, matlab-like library that automatically
  manages details like figure creation.

- via its internal object-oriented structure, that offers full control over all
  aspects of the figure, at the cost of slightly more verbose calls for the
  common case.

The pyplot api:

- Easiest to use.
- Sufficient for simple and moderately complex plots.
- Does not offer complete control over all details.

A simple example:

.. plot::
   :include-source:

   # Make a simple plot
   x = np.linspace(0, 2*np.pi)
   y = np.sin(x)
   plt.plot(x,y, label='sin(x)')
   plt.legend()
   plt.title('Harmonic')
   plt.xlabel('x')
   plt.ylabel('y')

   # Add one line to that plot
   z = np.cos(x)
   plt.plot(x, z, label='cos(x)')

   # Make a second figure with a simple plot
   plt.figure()
   plt.plot(x, np.sin(2*x), label='sin(2x)')
   plt.legend()

   # Other than when working interactively, in all real scripts you must
   # call show() at the end
   plt.show()


Here is how to create the same two plots, using explicit management of the
figure and axis objects:

.. plot::
   :include-source:
   
   # Array creation is the same
   x = np.linspace(0, 2*np.pi)
   y = np.sin(x)
   z = np.cos(x)
   
   f = plt.figure()  # we manually make a figure
   ax = f.add_subplot(111) # note 1-indexing
   ax.plot(x,y, label='sin(x)')  # it's the axis who plots
   ax.legend()
   ax.set_title('Harmonic')  # we set the title on the axis
   ax.set_xlabel('x')  # same with labels
   ax.set_ylabel('y')

   # Make a second figure with a simple plot.  We can name the figure with a
   # different variable name as well as its axes, and then control each
   f1 = plt.figure()
   ax1 = f1.add_subplot(111)  # we can name 
   ax1.plot(x, np.sin(2*x), label='sin(2x)')
   ax1.legend()
   
   # Since we now have variables for each axis, we can add back to the first
   # figure even after making the second
   ax.plot(x, z, label='cos(x)')


It's important to understand the existence of these objects, even if you use
mostly the top-level pyplot calls most of the time.  Many things can be
accomplished in MPL with mostly pyplot and a little bit of tweaking of the
underlying objects.  We'll revisit the object-oriented API later.

Important commands to know about, and which matplotlib uses internally a lot::

   gcf()  # get current figure
   gca()  # get current axis


Making subplots
===============

The simplest command is::

  plt.subplot(111)  # or (1,1,1)

which is equivalent to::

  f = plt.figure()
  f.add_subplot(111)

but works with the currently active figure.

A regular plot grid can be created with the ``subplots`` command:

.. plot::
    :include-source:
    
    x = np.linspace(0, 2*np.pi, 400)
    y = np.sin(x**2)

    # Just a figure and one subplot
    f, ax = plt.subplots()
    ax.plot(x, y)
    ax.set_title('Simple plot')

    # Two subplots, unpack the output array immediately
    f, (ax1, ax2) = plt.subplots(1, 2, sharey=True)
    ax1.plot(x, y)
    ax2.scatter(x, y)
    
    # Put a figure-level title
    f.suptitle('Sharing Y axis')

And finally, an arbitrarily complex grid can be made with ``subplot2grid``:
   
.. plot::
   :include-source:
   
   f = plt.figure()
   ax1 = plt.subplot2grid((3,3), (0,0), colspan=3)
   ax2 = plt.subplot2grid((3,3), (1,0), colspan=2)
   ax3 = plt.subplot2grid((3,3), (1, 2), rowspan=2)
   ax4 = plt.subplot2grid((3,3), (2, 0))
   ax5 = plt.subplot2grid((3,3), (2, 1))

   # Let's turn off visibility of all tick labels here
   for ax in f.axes:
      for t in ax.get_xticklabels()+ax.get_yticklabels():
          t.set_visible(False)
	  
   # And add a figure-level title at the top
   f.suptitle('Subplot2grid')


Manipulating properties across matplotlib
=========================================

In matplotlib, most properties for lines, colors, etc, can be set directly in
the call:

.. plot::

    plt.plot([1,2,3], linestyle='--', color='r')

But for finer control you can get a hold of the returned line object (more on
these objects later)::

    In [1]: line, = plot([1,2,3])

These line objects have a lot of properties you can control, a full list is
seen here by tab-completing in IPython::

    In [2]: line.set
    line.set                     line.set_drawstyle           line.set_mec
    line.set_aa                  line.set_figure              line.set_mew
    line.set_agg_filter          line.set_fillstyle           line.set_mfc
    line.set_alpha               line.set_gid                 line.set_mfcalt
    line.set_animated            line.set_label               line.set_ms
    line.set_antialiased         line.set_linestyle           line.set_picker
    line.set_axes                line.set_linewidth           line.set_pickradius
    line.set_c                   line.set_lod                 line.set_rasterized
    line.set_clip_box            line.set_ls                  line.set_snap
    line.set_clip_on             line.set_lw                  line.set_solid_capstyle
    line.set_clip_path           line.set_marker              line.set_solid_joinstyle
    line.set_color               line.set_markeredgecolor     line.set_transform
    line.set_contains            line.set_markeredgewidth     line.set_url
    line.set_dash_capstyle       line.set_markerfacecolor     line.set_visible
    line.set_dashes              line.set_markerfacecoloralt  line.set_xdata
    line.set_dash_joinstyle      line.set_markersize          line.set_ydata
    line.set_data                line.set_markevery           line.set_zorder
    

But the ``setp`` call (short for set property) can be very useful, especially
while working interactively because it contains introspection support, so you
can learn about the valid calls as you work::

    In [7]: line, = plot([1,2,3])

    In [8]: setp(line, 'linestyle')
      linestyle: [ ``'-'`` | ``'--'`` | ``'-.'`` | ``':'`` | ``'None'`` | ``' '`` | ``''`` ]         and any drawstyle in combination with a linestyle, e.g. ``'steps--'``.         

    In [9]: setp(line)
      agg_filter: unknown
      alpha: float (0.0 transparent through 1.0 opaque)         
      animated: [True | False]         
      antialiased or aa: [True | False]
      ...
      ... much more output elided
      ...

In the first form, it shows you the valid values for the 'linestyle' property,
and in the second it shows you all the acceptable properties you can set on the
line object.  This makes it very easy to discover how to customize your figures
to get the visual results you need.

Furthermore, setp can manipulate multiple objects at a time:

.. plot::
    :include-source:

    from numpy import linspace, sin, pi
    x = linspace(0, 2*pi)
    y1 = sin(x)
    y2 = sin(2*x)
    lines = plt.plot(x, y1, x, y2)
    
    # We will set the width and color of all lines in the figure at once:
    plt.setp(lines, linewidth=2, color='r')

Finally, if you know what properties you want to set on a specific object, a
plain ``set`` call is typically the simplest form:

.. plot::
   :include-source:
   
   line, = plt.plot([1,2,3])
   line.set(lw=2, c='red',ls='--')

   
Understanding what matplotlib returns: lines, axes and figures
==============================================================

Lines
-----

In a simple plot:

.. ipython::

    In [2]: plt.figure()  # ensure a fresh figure
    
    @savefig psimple.png width=4in
    In [3]: plt.plot([1,2,3])
    Out[3]: [<matplotlib.lines.Line2D object at 0x9b74d8c>]

The return value of the plot call is a list of lines, which can be manipulated
further.  If you capture the line object (in this case it's a single line so we
use a one-element tuple):

.. ipython::

   @savefig p1.png width=4in
   In [4]: line, = plt.plot([1,2,3])

you can then manipulate it directly, as we've already seen:
   
.. ipython::
   
   @savefig p2.png width=4in
   In [5]: line.set_color('r')
   In [6]: plt.draw()


One line property that is particularly useful to be aware of is ``set_data``:

.. plot::
   :include-source:

   # Create a plot and hold the line object
   line, = plt.plot([1,2,3], label='my data')
   plt.grid()
   plt.title('My title')
   
   # ... later, we may want to modify the x/y data but keeping the rest of the
   # figure intact, with our new data:
   x = np.linspace(0, 1)
   y = x**2

   # This can be done by operating on the data object itself
   line.set_data(x, y)

   # Now we must set the axis limits manually. Note that we can also use xlim
   # and ylim to set the x/y limits separately.
   plt.axis([0,1,0,1])

   # Note, alternatively this can be done with:
   ax = plt.gca()  # get currently active axis object
   ax.relim()
   ax.autoscale_view()
   
   # as well as requesting matplotlib to draw
   plt.draw()

   
The next important component, axes
----------------------------------
   
The ``axis`` call above was used to set the x/y limits of the axis.  And in
previous examples we called ``.plot`` directly on axis objects.  Axes are the
main object that contains a lot of the user-facing functionality of matplotlib::

    In [15]: f = plt.figure()

    In [16]: ax = f.add_subplot(111)

    In [17]: ax.
    Display all 299 possibilities? (y or n)
    ax.acorr                                 ax.hitlist
    ax.add_artist                            ax.hlines
    ax.add_callback                          ax.hold
    ax.add_collection                        ax.ignore_existing_data_limits
    ax.add_line                              ax.images
    ax.add_patch                             ax.imshow
    
    ... etc.

Many of the commands in ``plt.<command>`` are nothing but wrappers around axis
calls, with machinery to automatically create a figure and add an axis to it if
there wasn't one to begin with.  The output of most axis actions that draw
something is a collection of lines (or other more complex geometric objects).

Enclosing it all, the figure
----------------------------

The enclosing object is the ``figure``, that holds all axes::

    In [17]: f = plt.figure()

    In [18]: f.add_subplot(211)
    Out[18]: <matplotlib.axes.AxesSubplot object at 0x9d0060c>

    In [19]: f.axes
    Out[19]: [<matplotlib.axes.AxesSubplot object at 0x9d0060c>]

    In [20]: f.add_subplot(212)
    Out[20]: <matplotlib.axes.AxesSubplot object at 0x9eacf0c>

    In [21]: f.axes
    Out[21]: 
    [<matplotlib.axes.AxesSubplot object at 0x9d0060c>,
     <matplotlib.axes.AxesSubplot object at 0x9eacf0c>]

The basic view of matplotlib is: a figure contains one or more axes, axes draw
and return collections of one or more geometric objects (lines, patches, etc).

For all the gory details on this topic, see the matplotlib `artist tutorial`_.

.. _artist tutorial: http://matplotlib.sourceforge.net/users/artists.html


Anatomy of a common plot
========================

Let's make a simple plot that contains a few commonly used decorations

.. plot::
   :include-source:

   f = plt.figure()
   ax = f.add_subplot(111)

   # Three simple polyniomials
   x = np.linspace(-1, 1)
   y1,y2,y3 = [x**i for i in [1,2,3]]

   # Plot each with a label (for a legend)
   ax.plot(x, y1, label='linear')
   ax.plot(x, y2, label='cuadratic')
   ax.plot(x, y3, label='cubic')
   # Make all lines drawn so far thicker
   plt.setp(ax.lines, linewidth=2)

   # Add a grid and a legend that doesn't overlap the lines
   ax.grid(True)
   ax.legend(loc='lower right')

   # Add black horizontal and vertical lines through the origin
   ax.axhline(0, color='black')
   ax.axvline(0, color='black')

   # Set main text elements of the plot
   ax.set_title('Some polynomials')
   ax.set_xlabel('x')
   ax.set_ylabel('p(x)')

   
Common plot types
=================

Error plots
-----------

First a very simple error plot

.. plot::
    :include-source:
    
    # example data
    x = np.arange(0.1, 4, 0.5)
    y = np.exp(-x)

    # example variable error bar values
    yerr = 0.1 + 0.2*np.sqrt(x)
    xerr = 0.1 + yerr

    # First illustrate basic pyplot interface, using defaults where possible.
    plt.figure()
    plt.errorbar(x, y, xerr=0.2, yerr=0.4)
    plt.title("Simplest errorbars, 0.2 in x, 0.4 in y")

Now a more elaborate one, using the OO interface to exercise more features.

.. plot::
    :include-source:

    # same data/errors as before
    x = np.arange(0.1, 4, 0.5)
    y = np.exp(-x)
    yerr = 0.1 + 0.2*np.sqrt(x)
    xerr = 0.1 + yerr

    fig, axs = plt.subplots(nrows=2, ncols=2)
    ax = axs[0,0]
    ax.errorbar(x, y, yerr=yerr, fmt='o')
    ax.set_title('Vert. symmetric')

    # With 4 subplots, reduce the number of axis ticks to avoid crowding.
    ax.locator_params(nbins=4)

    ax = axs[0,1]
    ax.errorbar(x, y, xerr=xerr, fmt='o')
    ax.set_title('Hor. symmetric')

    ax = axs[1,0]
    ax.errorbar(x, y, yerr=[yerr, 2*yerr], xerr=[xerr, 2*xerr], fmt='--o')
    ax.set_title('H, V asymmetric')

    ax = axs[1,1]
    ax.set_yscale('log')
    # Here we have to be careful to keep all y values positive:
    ylower = np.maximum(1e-2, y - yerr)
    yerr_lower = y - ylower

    ax.errorbar(x, y, yerr=[yerr_lower, 2*yerr], xerr=xerr,
			         fmt='o', ecolor='g')
    ax.set_title('Mixed sym., log y')


Logarithmic plots
-----------------

A simple log plot

.. plot::
   :include-source:

   x = np.linspace(-5, 5)
   y = np.exp(-x**2)

   f, (ax1, ax2) = plt.subplots(2, 1)
   ax1.plot(x, y)
   ax2.semilogy(x, y)

A more elaborate log plot using 'symlog', that treats a specified range as
linear (thus handling values near zero) and symmetrizes negative values:

.. plot::
   :include-source:
   
   x = np.linspace(-50, 50, 100)
   y = np.linspace(0, 100, 100)

   # Create the figure and axes
   f, (ax1, ax2, ax3) = plt.subplots(3, 1)

   # Symlog on the x axis
   ax1.plot(x, y)
   ax1.set_xscale('symlog')
   ax1.set_ylabel('symlogx')
   # Grid for both axes
   ax1.grid(True)
   # Minor grid on too for x
   ax1.xaxis.grid(True, which='minor')

   # Symlog on the y axis
   ax2.plot(y, x)
   ax2.set_yscale('symlog')
   ax2.set_ylabel('symlogy')

   # Symlog on both
   ax3.plot(x, np.sin(x / 3.0))
   ax3.set_xscale('symlog')
   ax3.set_yscale('symlog')
   ax3.grid(True)
   ax3.set_ylabel('symlog both')


Bar plots
---------

.. plot::
   :include-source:
   
   # a bar plot with errorbars
   import numpy as np
   import matplotlib.pyplot as plt

   N = 5
   menMeans = (20, 35, 30, 35, 27)
   menStd =   (2, 3, 4, 1, 2)

   ind = np.arange(N)  # the x locations for the groups
   width = 0.35       # the width of the bars

   fig = plt.figure()
   ax = fig.add_subplot(111)
   rects1 = ax.bar(ind, menMeans, width, color='r', yerr=menStd)

   womenMeans = (25, 32, 34, 20, 25)
   womenStd =   (3, 5, 2, 3, 3)
   rects2 = ax.bar(ind+width, womenMeans, width, color='y', yerr=womenStd)

   # add some
   ax.set_ylabel('Scores')
   ax.set_title('Scores by group and gender')
   ax.set_xticks(ind+width)
   ax.set_xticklabels( ('G1', 'G2', 'G3', 'G4', 'G5') )

   ax.legend( (rects1[0], rects2[0]), ('Men', 'Women') )


Scatter plots
-------------

The ``scatter`` command produces scatter plots with arbitrary markers.

.. plot::
    :include-source:

    from matplotlib import cm
    from numpy import linspace, exp, cos, pi

    t = linspace(0.0, 6*pi, 100)
    y = exp(-0.1*t)*cos(t)
    phase = t % 2*pi
    f = plt.figure()
    ax = f.add_subplot(111)
    ax.scatter(t, y, s=100*abs(y), c=phase, cmap=cm.jet)
    ax.set_ylim(-1,1)
    ax.grid()
    ax.axhline(0, color='k')

    
Exercise
--------

Consider you have the following data in a text file::

    #Station  Lat    Long   Elev 
    BIRA    26.4840 87.2670 0.0120
    BUNG    27.8771 85.8909 1.1910
    GAIG    26.8380 86.6318 0.1660
    HILE    27.0482 87.3242 2.0880
    ILAM    26.9102 87.9227 1.1810
    JIRI    27.6342 86.2303 1.8660
    NAMC    27.8027 86.7146 3.5230
    PHAP    27.5150 86.5842 2.4880
    PHID    27.1501 87.7645 1.1760
    RUMJ    27.3038 86.5482 1.3190
    SIND    27.2107 85.9088 0.4650
    THAK    27.5996 85.5566 1.5510
    TUML    27.3208 87.1950 0.3600
    LAZE    29.1403 87.5922 4.0110
    SAJA    28.9093 88.0209 4.3510
    ONRN    29.3020 87.2440 4.3500
    SSAN    29.4238 86.7290 4.5850
    SAGA    29.3292 85.2321 4.5240
    DINX    28.6646 87.1157 4.3740
    RBSH    28.1955 86.8280 5.1000
    NAIL    28.6597 86.4126 4.3780
    MNBU    28.7558 86.1610 4.5000
    NLMU    28.1548 85.9777 3.8890
    YALA    28.4043 86.1133 4.4340
    XIXI    28.7409 85.6904 4.6600
    RC14    29.4972 86.4373 4.7560
    MAZA    28.6713 87.8553 4.3670
    JANA    26.7106 85.9242 0.0770
    SUKT    27.7057 85.7611 0.7450

These are the names of seismographic stations in the Himalaya, with their
coordinates and elevations in Kilometers.

Make a scatter plot of all of these, using both the size and the color to
(redundantly) encode elevation.


Histograms
----------

Matplotlib has a built-in command for histograms.

.. plot::
   :include-source:

   mu, sigma = 100, 15
   x = mu + sigma * np.random.randn(10000)

   # the histogram of the data
   n, bins, patches = plt.hist(x, 50, normed=1, facecolor='g', alpha=0.75)

   plt.xlabel('Smarts')
   plt.ylabel('Probability')
   plt.title('Histogram of IQ')
   plt.text(60, .025, r'$\mu=100,\ \sigma=15$')
   plt.axis([40, 160, 0, 0.03])
   plt.grid(True)



Aribitrary text and LaTeX support
=================================

In matplotlib, text can be added either relative to an individual axis object
or to the whole figure.

These commands add text to the Axes:

- title() - add a title
- xlabel() - add an axis label to the x-axis
- ylabel() - add an axis label to the y-axis
- text() - add text at an arbitrary location
- annotate() - add an annotation, with optional arrow

And these act on the whole figure:

- figtext() - add text at an arbitrary location
- suptitle() - add a title

And any text field can contain LaTeX expressions for mathematics, as long as
they are enclosed in ``$`` signs.

This example illustrates all of them:

.. plot::
   :include-source:

   fig = plt.figure()
   fig.suptitle('bold figure suptitle', fontsize=14, fontweight='bold')

   ax = fig.add_subplot(111)
   fig.subplots_adjust(top=0.85)
   ax.set_title('axes title')

   ax.set_xlabel('xlabel')
   ax.set_ylabel('ylabel')

   ax.text(3, 8, 'boxed italics text in data coords', style='italic',
	   bbox={'facecolor':'red', 'alpha':0.5, 'pad':10})

   ax.text(2, 6, r'an equation: $E=mc^2$', fontsize=15)

   ax.text(3, 2, unicode('unicode: Institut f\374r Festk\366rperphysik',
                         'latin-1'))

   ax.text(0.95, 0.01, 'colored text in axes coords',
	   verticalalignment='bottom', horizontalalignment='right',
	   transform=ax.transAxes,
	   color='green', fontsize=15)


   ax.plot([2], [1], 'o')
   ax.annotate('annotate', xy=(2, 1), xytext=(3, 4),
	       arrowprops=dict(facecolor='black', shrink=0.05))

   ax.axis([0, 10, 0, 10])


Axis sharing
============

The simplest way to share axes is to use the ``subplots`` function.  More
fine-grained control can be obtained by individually adding subplots and adding
share calls to those, but in most cases the functionality from ``subplots`` is sufficient:

.. plot::
   :include-source:
   
   # Simple data to display in various forms
   x = np.linspace(0, 2*np.pi, 400)
   y = np.sin(x**2)

   # Two subplots, the axes array is 1-d
   f, axarr = plt.subplots(2, sharex=True)
   f.suptitle('Sharing X axis')
   axarr[0].plot(x, y)
   axarr[1].scatter(x, y)

   # Two subplots, unpack the axes array immediately
   f, (ax1, ax2) = plt.subplots(1, 2, sharey=True)
   f.suptitle('Sharing Y axis')
   ax1.plot(x, y)
   ax2.scatter(x, y)

   # Three subplots sharing both x/y axes
   f, (ax1, ax2, ax3) = plt.subplots(3, sharex=True, sharey=True)
   f.suptitle('Sharing both axes')
   ax1.plot(x, y)
   ax2.scatter(x, y)
   ax3.scatter(x, 2*y**2-1,color='r')
   # Fine-tune figure; make subplots close to each other and hide x ticks for
   # all but bottom plot.
   f.subplots_adjust(hspace=0)
   plt.setp([a.get_xticklabels() for a in f.axes[:-1]], visible=False)

   
Basic event handling
====================

Matplotlib has a builtin, toolkit-independent event model that is fairly rich.
If you want to develop full-fledged applications with very complex and fast
interactions, you are likely better off choosing a specific Graphical User
Interface (GUI) toolkit and using its specific event model.  But for many
scientific uses, what matplotlib offers is more than sufficient, and it has the
advantage of working identically regardless of the GUI toolkit you choose to
run matplotlib under.

Here we will cover the bare essentials only, for full details you should
consult the `event handling section`_ of the matplotlib user guide.

.. _event handling section: http://matplotlib.sourceforge.net/users/event_handling.html

The basic idea of *all* event handling is always the same: the windowing
environment registers an event (mouse movement, click, keyboard press, etc)
produced by the user.  In advance, you have registered *event handlers*:
functions you define that are meant to be called when specific types of events
occur.  The registration action is called *connecting* the event handler, and
is performed by the ``mpl_connect`` method of the figure canvas attribute (the
canvas is the drawing area of the figure object, the entire raw object where
events take place).

The windowing system will then pass the event (each event has some relevant
information that goes with it, such as which key or button was pressed) to your
function, which can act on it.  These functions are referred to as *callbacks*,
because they are meant to be 'called back' not by you, but by the windowing
toolkit when the right event goes by.

Here is the simplest possible matplotlib event handler::

    fig = plt.figure()
    ax = fig.add_subplot(111)
    ax.plot(np.random.rand(10))

    def onclick(event):
	print 'button=%d, x=%d, y=%d, xdata=%f, ydata=%f'%(
	    event.button, event.x, event.y, event.xdata, event.ydata)

    cid = fig.canvas.mpl_connect('button_press_event', onclick)

The ``FigureCanvas`` method ``mpl_connect`` returns a connection id which
is simply an integer.  When you want to disconnect the callback, just call::

    fig.canvas.mpl_disconnect(cid)

The most commonly used event types are ``KeyEvent`` and ``MouseEvent``, both of
which have the following attributes:

    ``x``
        x position - pixels from left of canvas

    ``y``
        y position - pixels from bottom of canvas

    ``inaxes``
        the ``matplotlib.axes.Axes`` instance if mouse is over axes

    ``xdata``
        x coord of mouse in data coords

    ``ydata``
        y coord of mouse in data coords

In addition, ``MouseEvent`` have:

    ``button``
        button pressed None, 1, 2, 3, 'up', 'down' (up and down are used for
        scroll events)

    ``key``
        the key pressed: None, any character, 'shift', 'win', or 'control'

	
Exercise
--------

Extend the scatter plot exercise above, to print the location (four-letter
string) of the station you click on.  Use a threshold for distance, and
discriminate between a click below threshold (considered to be 'on') vs a miss,
in which case you should indicate what the closest station is, its coordinates
and the distance to it from the click.

Image display
=============

The ``imshow`` command can display single or multi-channel images.  A simple
array of random numbers, plotted in grayscale:

.. plot::
   :include-source:

   from matplotlib import cm
   plt.imshow(np.random.rand(128, 128), cmap=cm.gray, interpolation='nearest')

A real photograph is a multichannel image, imwhow interprets it correctly:
   
.. plot::
   :include-source:

   img = plt.imread('data/stained_glass_barcelona.png')
   plt.imshow(img)

   
Exercise
--------

1. Display each color channel of the image with the right colormap.
2. Display a luminosity colormap as well as one for each color channel.
3. Make a plot of the image plus all three color channels in the same figure,
   so that zooming into one zooms into all the others.

Hint: look for the matplotlib image tutorial.


Simple 3d plotting with matplotlib
==================================

Note that you must execute at least once in your session::

  from mpl_toolkits.mplot3d import Axes3D

One this has been done, you can create 3d axes with the ``projection='3d'``
keyword to ``add_subplot``::

  fig = plt.figure()
  fig.add_subplot(<other arguments here>, projection='3d')

A simple surface plot:

.. plot::
   :include-source:
   
   from mpl_toolkits.mplot3d.axes3d import Axes3D
   from matplotlib import cm
   
   fig = plt.figure()
   ax = fig.add_subplot(1, 1, 1, projection='3d')
   X = np.arange(-5, 5, 0.25)
   Y = np.arange(-5, 5, 0.25)
   X, Y = np.meshgrid(X, Y)
   R = np.sqrt(X**2 + Y**2)
   Z = np.sin(R)
   surf = ax.plot_surface(X, Y, Z, rstride=1, cstride=1, cmap=cm.jet,
	   linewidth=0, antialiased=False)
   ax.set_zlim3d(-1.01, 1.01)

And a parametric surface specified in cylindrical coordinates:
   
.. plot::
   :include-source:

   from mpl_toolkits.mplot3d import Axes3D

   matplotlib.rcParams['legend.fontsize'] = 10

   fig = plt.figure()
   ax = fig.add_subplot(111, projection='3d')
   theta = np.linspace(-4*np.pi, 4*np.pi, 100)
   z = np.linspace(-2, 2, 100)
   r = z**2 + 1
   x = r*np.sin(theta)
   y = r*np.cos(theta)
   ax.plot(x, y, z, label='parametric curve')
   ax.legend()
   