+++
title = "Analysing the weather in Trieste, Italy"
subtitle = ""

# Add a summary to display on homepage (optional).
summary = "Using public data from the Molo Bandiera weather station to produce nice plots"

date = 2019-05-05T14:13:29+02:00
draft = false

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ['Adriano Fantini']

# Is this a featured post? (true/false)
featured = false

# Tags and categories
# For example, use `tags = []` for no tags, or the form `tags = ["A Tag", "Another Tag"]` for one or more tags.
tags = ['R', 'weather', 'dplyr', 'ggplot2', 'Trieste', 'patchwork']
categories = ['R', 'weather']

# Code highlighting
highlight = true
highlight_languages = ["r"]

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["deep-learning"]` references 
#   `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
# projects = ["internal-project"]

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder. 
[image]
  # Caption (optional)
  caption = ""

  # Focal point (optional)
  # Options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
  focal_point = ""

[[gallery_item]]
  album = "post-weather-trieste"
  image = "post/weather-trieste/Temperature.png"
  caption = "Temperature"
[[gallery_item]]
  album = "post-weather-trieste"
  image = "post/weather-trieste/Humidity.png"
  caption = "Humidity"
[[gallery_item]]
  album = "post-weather-trieste"
  image = "post/weather-trieste/Pressure.png"
  caption = "Pressure"
[[gallery_item]]
  album = "post-weather-trieste"
  image = "post/weather-trieste/Radiation.png"
  caption = "Radiation"
[[gallery_item]]
  album = "post-weather-trieste"
  image = "post/weather-trieste/Wind.png"
  caption = "Wind"
[[gallery_item]]
  album = "post-weather-trieste"
  image = "post/weather-trieste/WindMax.png"
  caption = "Wind gusts"
[[gallery_item]]
  album = "post-weather-trieste"
  image = "post/weather-trieste/WindDir.png"
  caption = "Wind direction"
[[gallery_item]]
  album = "post-weather-trieste"
  image = "post/weather-trieste/WindMaxDir.png"
  caption = "Wind gusts direction"
[[gallery_item]]
  album = "post-weather-trieste"
  image = "post/weather-trieste/Rain.png"
  caption = "Precipitation"
+++

Let's analyse last month's weather in Trieste, Italy, where I live. I was away for most of the month, so I do not know much of what happened, and I am curious.

Trieste is a coastal city well known for its mild climate and strong winds, especially a strong, cold, north-easterly wind called the _Bora_ which can easily reach above 100km/h and strikes a few times per year.  
We'll use data coming from the regional environmental agency, which can be obtained from [here](http://www.osmer.fvg.it/archivio.php?ln=&p=dati) by selecting:

- select _anno_ (year) and _mese_ (month)
- _giorno_ (day) -> tutti (all)
- _stazione_ (station) -> Trieste Molo F.lli Bandiera
- _orari_ (hourly)
- _visualizza dati_ (retrieve data)
- scroll to the end of the page, and click _salva come csv_ (save as csv)

{{% alert note %}}
See [this](https://github.com/adrfantini/lastmonth-weather) GitHub repo for the complete code for this blog post!
{{% /alert %}}

The `.csv` file we'll get contains all the data from this station, which has several sensors. We'll use only R to read and plot the data, primarily via some packages in the [`tidyverse`](https://www.tidyverse.org/). Let's first load the packages that we need.

```R
library(ggplot2) # plots
library(dplyr) # data wrangling
library(RColorBrewer) # additional colors
library(lubridate) # dealing with time
```

I prepare these plots periodically, so I automate the selection of the last completed month:

```R
current_date = Sys.Date()
year = format(current_date, "%Y")
month = format(current_date, "%m")
basepath = "csv/dati-orari-Trieste molo F.lli Bandiera-"

last_month = (as.numeric(month) - 1) %>% sprintf(fmt = '%02d')
```

Now open the `.csv` file, removing unwanted characters and three empty lines at the end:

```R
fn = paste0(basepath, year, last_month, ".csv")
csv = read.csv(fn, sep=";", stringsAsFactors=FALSE, na.strings = c("NA", "-", " - ", "NaN", "", "/", "NAN", "NULL"))
csv = csv[1:(nrow(csv)-3),]
```

This data source is already in [_tidy format_](https://cran.r-project.org/web/packages/tidyr/vignettes/tidy-data.html): it has a single row per observation and a single column per variable. This comes in handy: we just need to rename variables to something more appropriate, apply minor tweaks, and we are good to go.
We also remove one column (`Bagnatura.Fogliare.min.`) which does not contain data.

```R
d = csv %>% select(-Bagnatura.Fogliare.min.) %>%
    setNames(c("Day", "Hour", "Rain", "Temperature", "Humidity", "Wind", "WindDir", "WindMax", "WindMaxDir", "Radiation", "Pressure")) %>%
    mutate(Month = month.name[as.numeric(last_month)], Year = as.numeric(year), Day = as.numeric(Day))

d[d$Hour == 0, "Hour"] = 24

d$Rain[d$Rain == 0] = NA

d$Hour = factor(sprintf("%02d", d$Hour), levels = sprintf("%02d", 1:24))
```

We prepare now to add a small little cool addon to our plots: sunrise and sunset lines. We approx these with the `sunriset` function from the `maptools` package, passing the location as lat-lon and the days of the last month as `POSIXct` objects:

```R
location = c(13.7516938, 45.649985)
startdate = as.Date(paste(year, last_month, 1), "%Y %m %d") # First day of the month
last_month_days = startdate + 1:days_in_month(startdate) - 1 # All days in the last month
sunrise = maptools::sunriset(matrix(location, nrow=1), as.POSIXct(last_month_days, tz="UTC"), POSIXct.out=TRUE, direction="sunrise")$time
sunset = maptools::sunriset(matrix(location, nrow=1), as.POSIXct(last_month_days, tz="UTC"), POSIXct.out=TRUE, direction="sunset")$time
sun_times = data.frame(
    day = day(last_month_days),
    time = c(hour(sunrise) + minute(sunrise) / 60, hour(sunset) + minute(sunset) / 60),
    group = rep(c("Sunrise", "Sunset"), each = length(last_month_days))
)
```

We want to plot the following:

- An x-y colored table with overlayed wind arrows
- Sunrise / sunset lines
- A line between Sunday and Monday to separate weeks
- A caption
- Different color scales for different variables
- Overlay wind arrows (spokes) for each moment

We'll see how I covered all of this in a single function.

Unfortunately, `ggplot` can be quite hard to get some details right (especially the positionment of legends, titles etc.), and the following has been tweaked to work well to a default `ggplot` size output (7x7 inches). If you want to plot at different sizes, you might need to adjust some constants, some of which are in the `theme`:

```R
mytheme = theme(
    panel.background = element_blank(),
    panel.grid = element_blank(),
    axis.ticks = element_blank(),
    axis.text.y = element_text(size = 12, colour = "grey20"),
    axis.text.x = element_text(size = 12, colour = "grey20"),
    plot.title = element_text(lineheight=1.2, face="bold", size = 16, colour = "grey20", margin = margin(b = 15, unit = "pt")),
    plot.subtitle = element_text(colour = "grey20", margin = margin(b = 15, unit = "pt")),
    legend.title = element_blank(),
    legend.direction = "horizontal",
    legend.box = "horizontal",
    legend.key = element_blank(),
    legend.key.width = unit(1.2,"cm"),
    legend.position = c(1,1.09),
    legend.justification = c(1,1)
)
```

Lets start. We find out when new weeks are starting:

```R
weekline = which(weekdays(last_month_days) == "Sunday") + 0.5
```

Then we create a base plot which will be used by all subsequent plots. Here we define our X (hour of day) and Y axis (day),

```R
day_initials = substr(weekdays(last_month_days), 0, 1)
day_labels = paste(day_initials, day(last_month_days))
base_plot = ggplot(d, aes(x = Hour, y = Day)) +
        xlab("HOUR OF DAY (UTC)") +
        ylab("DAY") +
        scale_x_discrete(expand = c(0, 0)) +
        scale_y_reverse(breaks = day(last_month_days), labels = day_labels, expand = c(0, 0)) +
        coord_cartesian(xlim=c(0.5, 24.5), ylim = c(0.5, length(last_month_days) + 0.5)) +
        mytheme
```

And then we define the plotting function. I commented where appropriate.

```R
monthPlot = function(
        d,
        column_to_plot,
        palette,
        direction = 1, # Color direction for RCOlorBrewer palettes
        breaks,
        labels = breaks,
        values = NULL,
        title = column_to_plot,
        caption = paste("Data for Molo Fratelli Bandiera, Trieste, Italy;", month.name[as.numeric(last_month)], year, "
        Data source: ARPA FVG, OSMER and GRN – www.meteo.fvg.it       Produced by: Adriano Fantini  – www.adrianofantini.eu"),
        na.value = NA,
        show_suntime = TRUE,
        show_weekline = TRUE,
        show_windspokes = TRUE,
        wind_scaling = 1/40,
        wind_arrow_length = 0.15,
        wind_arrow_thickness = 0.2
    ) {
    if (palette %in% rownames(brewer.pal.info)) { # Get colors: use RColorBrewer if appropriate
        colors = brewer.pal(9, palette)
        if (direction == -1) colors = rev(colors)
    } else { # Else use custom colors
        colors = palette
    }

    bp = base_plot + # Set what we want to plot, title and caption
        geom_tile(
            aes_string(fill = column_to_plot),
            colour = "grey70"
        ) +
        labs(title = title, caption = caption)

    if (isTRUE(show_windspokes)) { # Show wind spokes, using some scaling values for the lengths
        bp = bp +
            geom_spoke(aes(
                angle = -(WindDir/360*2*pi) - pi/2,
                radius = Wind * wind_scaling
            ),
            arrow = arrow(length = unit(wind_arrow_length,"cm")),
            alpha = 0.8,
            size = wind_arrow_thickness)
    }

    if (isTRUE(show_weekline)) { # Show week line as an hline
        bp = bp +
            geom_hline(yintercept = weekline)
    }

    if (isTRUE(show_suntime)) { # Show two lines for sunrise and sunset, and tweak the legend for it
    bp = bp +
        geom_line(data = sun_times, aes(
            x = time,
            y = day,
            group = group,
            color = "Sunrise\nSunset"
        ), size = 1.5, alpha = 0.8, lineend = "round") +
        scale_color_manual(values="grey37") +
        guides(
            fill = guide_colourbar(order = 2),
            color = guide_legend(order = 1)
        )
    }

    if (!missing(breaks)) { # Set colors
        bp + scale_fill_gradientn(
            colors = colors,
            values = values,
            breaks = breaks,
            labels = labels,
            limits = range(breaks),
            oob = scales::squish,
            na.value = na.value
        )
    } else {
        bp + scale_fill_gradientn(colors = colors, na.value = NA)
    }
}
```

Now nothing's left but plotting, choosing appropriate colors for each variable.

```R
dir.create('lastmonth_plots', recursive = TRUE, showWarnings = FALSE)

p1 = d %>%
    monthPlot("Rain", "YlGnBu", title = "Rain [mm]")
ggsave(plot = p1, filename = "lastmonth_plots/Rain.png")

p2 = d %>%
    monthPlot("Temperature", c(rev(brewer.pal(6, "Spectral")), "#9e0098"), title = "Temperature [°C]")
ggsave(plot = p2, filename = "lastmonth_plots/Temperature.png")

p3 = d %>%
    monthPlot("Humidity", "BrBG", title = "Humidity [%]")
ggsave(plot = p3, filename = "lastmonth_plots/Humidity.png")

p4 = d %>%
    monthPlot("Wind", "Oranges", title = "Wind speed [km/h]")
ggsave(plot = p4, filename = "lastmonth_plots/Wind.png")

p5 = d %>%
    monthPlot("WindMax", "Oranges", title = "Wind gusts [km/h]")
ggsave(plot = p5, filename = "lastmonth_plots/WindMax.png")

p6 = d %>%
    monthPlot("Radiation", c("#282a9b", "#fff321", "#ffa500", "#e50909"), title = "Radiation [kJ/m2]")
ggsave(plot = p6, filename = "lastmonth_plots/Radiation.png")

p7 = d %>%
    monthPlot("Pressure", c(rev(brewer.pal(6, "Spectral")), "#9e0098"), title = "Pressure [hPa]")
ggsave(plot = p7, filename = "lastmonth_plots/Pressure.png")
```

Wind directions are a bit more complex if we want to show N-S-W-E labels, but they just require some tweaking to the breaks:

```R
wind_dir_colors = c("#e41a1c", "#377eb8", "#4daf4a", "#ffffb2")
wind_dir_colors = c(rep(wind_dir_colors, each=2)[8], rep(wind_dir_colors, each=2)[-8])
p8 = d %>%
    monthPlot("WindDir", wind_dir_colors, title = "Wind direction", breaks = c(0,90,180,270,360), labels = c("N", "E", "S", "W", "N"), values = scales::rescale(c(0,90,91,180,181,270,271,360)))
ggsave(plot = p8, filename = "lastmonth_plots/WindDir.png")

p9 = d %>%
    monthPlot("WindMaxDir", wind_dir_colors, title = "Wind gusts direction", breaks = c(0,90,180,270,360), labels = c("N", "E", "S", "W", "N"), values = scales::rescale(c(0,90,91,180,181,270,271,360)))
ggsave(plot = p9, filename = "lastmonth_plots/WindMaxDir.png")
```

And here are the results: click on the images to enlarge. Notice that we had 5 days of strong winds between the 11th and the 15th: in those days the wind direction was predominantly from the North-East, so the wind was _bora_.  
It was followed by a bout of good weather and high pressure, which ended with a bit of rain on the 23rd (least sunny day of the month).  
Also, interestingly, we have winds predominantly from the sea (West, greens) during the day, and from the land (East, reds) during the night. This is the well--known [sea breeze](https://en.wikipedia.org/wiki/Sea_breeze) effect.

{{< gallery album="post-weather-trieste" >}}

---

As an added bonus, we can calculate the correlation between the different variables, even though that's just on a very small time scale, so not very robust. We use only packages from the `tidyverse` (`tibble`, `tidyr`, `dplyr`):

```R
pcorr = d %>%
    select(-Day, -Hour, -Month, -Year) %>%
    cor(use = "na.or.complete") %>%
    as.data.frame() %>%
    tibble::rownames_to_column(var = "Variable1") %>%
    tidyr::gather(Rain, Temperature, Pressure, Humidity, Wind, WindDir, WindMax, WindMaxDir, Radiation, key = "Variable2", value = "Correlation") %>%
    filter(Correlation != 1) %>%
    ggplot(aes(x = Variable1, y = Variable2, fill = Correlation, label = round(Correlation, 2))) +
        geom_tile(color="grey70") +
        scale_fill_distiller(palette = "Spectral", limits = c(-1,1), direction = 1) +
        geom_text() +
        theme(
            axis.title = element_blank(),
            legend.title = element_blank(),
            legend.direction = "horizontal",
            legend.box = "horizontal",
            legend.key = element_blank(),
            legend.key.width = unit(1.2,"cm"),
            legend.position = c(1,1.09),
            legend.justification = c(1,1),
            plot.title = element_text(lineheight=1.2, face="bold", size = 16, colour = "grey20", margin = margin(b = 15, unit = "pt"))
        ) +
        ggtitle("Variable correlation") +
        labs(caption = paste("Data for Molo Fratelli Bandiera, Trieste, Italy;", month.name[as.numeric(last_month)], year, "
        Data source: ARPA FVG, OSMER and GRN – www.meteo.fvg.it       Produced by: Adriano Fantini  – www.adrianofantini.eu")) +
        coord_cartesian(expand=c(0,0))

ggsave(plot=pcorr, filename = "lastmonth_plots/correlation.png", w=7, h=7.2)
```

<img src="/img/post/weather-trieste/correlation.png" style="width:70%;" title="Correlation between weather variables in Trieste, April 2019">

As expected, some variables are strongly correlated (max wind and average wind speeds, or humidity and rain) while others are anti-correlated (humidity and average wind speed). However, doing this same analysis on a much larger timescale would probably lead to more interesting and robust results: I would expect for example temperature to be positively correlated with radiation.

Note that the correlations with wind direction variables are not really that simple to interpret, since they are periodic variables where `0==360`.

---

Using the nice package [`patchwork`](https://github.com/thomasp85/patchwork) we can put these plots together in a single image.
It's a nice showcase of `patchwork`'s simplicity and elegance:

```R
library(patchwork)
pptheme = theme(
    plot.title = element_text(face="bold", size = 22, colour = "grey20", hjust = 0.5)
)
# Order plots by your preference
pp3 = p7 + p3 + p8 + p9 + p1 + p6 + p2 + p4 + p5 + pcorr + plot_layout(ncol = 5) +
    plot_annotation(
        title = paste(month.name[as.numeric(last_month)], year, 'weather in Molo Fratelli Bandiera, Trieste, Italy'),
        theme = pptheme,
        caption = "Data source: ARPA FVG, OSMER and GRN – www.meteo.fvg.it
                 Produced by: Adriano Fantini  – www.adrianofantini.eu"
        ) &
    labs(caption = NULL) # Remove caption from all plots, since we have a global caption
ggsave(plot = pp3, filename = "lastmonth_plots/patchwork.png", w = 35, h = 15, dpi = 200)
```

<img src="/img/post/weather-trieste/patchwork.png" title="Weather in Trieste, April 2019">

Ta-da!
