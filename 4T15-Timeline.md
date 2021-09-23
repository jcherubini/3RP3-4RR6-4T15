# Biochem 4T15 Undergraduate Thesis Timeline :calendar:

This is a sample timeline for your progress as a thesis student in the VDL. Included in the timeline are key milestones to achieve throughout the academic year.
There are also some optional (but encouraged) conferences for you to consider presenting at and attending. As a sample timeline, the dates and milestones are
subjected to change based on your preferences and the learning goals that you have defined as a thesis student. You're encouraged to work with your supervisor to
customize your timeline!

<p align="center">
  <img src="https://github.com/jcherubini/3RP3-4RR6-4T15/blob/main/Figures/4t15-timeline.png" width="1100" height="500">
</p>
<p align="center">
</p>

-------

:pushpin: The code to reproduce and amend the timeline is posted below. Please work with your supervisor to include any other intersting dates, events, or milestones that 
you would like to strive towards.

```
# 4T15 timeline for 2021/22 academic year
## Adapted from Ben Alex Keen: https://benalexkeen.com/creating-a-timeline-graphic-using-r-and-ggplot2/.

library(ggplot2)
library(scales)
library(lubridate)
library(tidyverse)

df <- tibble(Month = c("09", "10", "10", "11", "12", "01", "02", "03", "04", "05"), Year = c(rep(2021, 5), rep(2022, 5)), Day = c(rep(1, 1), rep(10, 1), rep(29, 1), rep(15, 7)), Milestone = c("Topic Submission", "Outline and Reference List", "Problem statement", "Feedback Review", "Final Literature Review", "Begin data collection", "OEP Conference: Proposal / preliminary data", "Data collectoin", "Data collection / analysis", "NURC Conference"), Status = c("Complete", "Complete", "Complete", "In progress", "In progress", "In progress", "In progress", "In progress", "In progress", "In progress"))

df <- df %>%
  mutate(date = make_date(Year, Month, Day))

status_levels <- c("Complete", "In progress")
status_colors <- c("#00B050", "#0070C0")

df$Status <- factor(df$Status, levels = status_levels, ordered = TRUE)

positions <- c(0.5, -0.5, 1.0, -1.0, 1.5, -1.5)
directions <- c(1, -1)

line_pos <- data.frame(
    "date"=unique(df$date),
    "position"=rep(positions, length.out=length(unique(df$date))),
    "direction"=rep(directions, length.out=length(unique(df$date)))
  )
df <- merge(x=df, y=line_pos, by="date", all = TRUE)
df <- df[with(df, order(date, Status)), ]

text_offset <- 0.05

df$month_count <- ave(df$date==df$date, df$date, FUN=cumsum)
df$text_position <- (df$month_count * text_offset * df$direction) + df$position

month_buffer <- 2

month_date_range <- seq(min(df$date) - months(month_buffer), max(df$date) + months(month_buffer), by='month')
month_format <- format(month_date_range, '%b')
month_df <- data.frame(month_date_range, month_format)

year_date_range <- seq(min(df$date) - months(month_buffer), max(df$date) + months(month_buffer), by='year')
year_date_range <- as.Date(
  intersect(
    ceiling_date(year_date_range, unit="year"),
    floor_date(year_date_range, unit="year")
  ),  origin = "1970-01-01"
)

year_format <- format(year_date_range, '%Y')
year_df <- data.frame(year_date_range, year_format)

# Plot

timeline_plot <- ggplot(df, aes(x = date, y=0, col=Status, label = "Milestone"))
timeline_plot <- timeline_plot +
  labs(col = "Milestones")
timeline_plot <- timeline_plot +
  scale_color_manual(values = status_colors, labels = status_levels, drop = FALSE)
timeline_plot <- timeline_plot +
  theme_classic()

# Plot horizontal black line for timeline
timeline_plot <- timeline_plot +
  geom_hline(yintercept = 0, color = "black", size = 0.75)

# Plot vertical segment lines for milestones
timeline_plot <- timeline_plot +
  geom_segment(data = df[df$month_count == 1,], aes(y = position, yend = 0, xend = date), color = 'black', size = 0.2)

# Plot scatter points at zero and date
timeline_plot <- timeline_plot +
  geom_point(aes(y = 0), size = 3)

# Don't show axes, appropriately position legend
timeline_plot <- timeline_plot +
  theme(axis.line.y = element_blank(),
        axis.text.y = element_blank(),
        axis.title.x = element_blank(),
        axis.title.y = element_blank(),
        axis.ticks.y = element_blank(),
        axis.text.x = element_blank(),
        axis.ticks.x = element_blank(),
        axis.line.x = element_blank(),
        legend.position = "bottom")

# Show text for each month
timeline_plot <- timeline_plot +
  geom_text(data = month_df, aes(x = month_date_range, y = -0.2, label = month_format, fontface= "bold"), size = 3.5, vjust = 0.5, color ='black', angle = 45)

# Show year text
timeline_plot <- timeline_plot +
  geom_text(data = year_df, aes(x = year_date_range, y = -0.48, label = year_format, fontface = "bold"), size = 4.2, color = 'black')

# Show text for each milestone
timeline_plot <- timeline_plot +
  geom_text(data = df, aes(y = text_position, label = Milestone, fontface = "bold"), size = 3)
timeline_plot
```
