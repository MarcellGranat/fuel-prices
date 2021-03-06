Fuel prices - results
================

## Casuality map

``` mermaid
graph LR
A[üzemanyag ár]
B(Világpiaci olajár) --> A
C(Utak hossza)-->A
D(Gépjárművek száma)-->A
E(Autópálya)-.->E1(Melette van?)
E-..->E2(Távolság a csomóponttól)
E1-->A
E2-->A
F(Márka)-..->G[Kategória]
G-->A
H(Százhalombattától vett távolság)-->A
```

``` r
source("codes/utils.R")
load("data/fuel_df.RData")
load("data/city_df.RData")
load("data/energy_agency_df.RData")
load("data/crude_df.RData")

theme_set(
  theme_bw() + 
    theme(
      legend.position = "bottom"
    )
)
```

``` r
energy_agency_df %>% 
  pivot_longer(-1, names_to = "fuel", values_to = "price") %>% 
  ggplot() + 
  geom_vline(aes(xintercept = as.Date("2021-11-15"), linetype = "Beginning of price control (2021-11-15)"), color = "orange", size = 1.5) +
  geom_line(aes(time, price, color = fuel)) +
  geom_point(aes(time, price, color = fuel), show.legend = FALSE) + 
  geom_point(data = count(fuel_df, date), aes(x = date, y = 520, fill = "Data sucessfully collected"), shape = 21, color = "purple", alpha = .5) +
  scale_fill_manual(values = "purple") +
  scale_linetype_manual(values = 2) +
  scale_x_date(breaks = scales::date_breaks("months"), labels = ~ NiceMonth(., keep_year = TRUE, label = TRUE)) + 
  labs(fill = NULL, linetype = NULL, color = "Fuel type", x = NULL, y = "Reported average fuel price (Hungrian Forint)")
```

<img src="figures/price_control-1.png" style="display: block; margin: auto;" />

``` r
p1 <- crude_df %>% 
  mutate(price_huf = crude_price * usd_huf) %>% 
  pivot_longer(-1) %>% 
  mutate(name = fct_inorder(name)) %>% 
  na.omit() %>% 
  ggplot(aes(time, value)) +
  facet_wrap(~ name, scales = "free_y", ncol = 1, labeller = as_labeller(c("usd_huf" = "US dollar in HUF", "crude_price" = "Crude oil price in US dollar", "price_huf" = "Crude oil price in HUF"))) + 
  geom_line() + 
  scale_x_date(breaks = scales::date_breaks("months"), labels = ~ NiceMonth(., keep_year = TRUE, label = TRUE)) + 
  labs(x = NULL, y = " ")

p1
```

<img src="figures/crude_price_raw-1.png" style="display: block; margin: auto;" />

``` r
p2 <- crude_df %>% 
  mutate(price_huf = crude_price * usd_huf) %>% 
  slice(-1) %>% 
  mutate_at(-1, ~ . / first(.)) %>% 
  pivot_longer(-1) %>%
  mutate(
    name = fct_inorder(name),
    name = fct_relabel(name, ~ case_when(
      . == "usd_huf" ~ "US dollar in HUF", 
      . == "crude_price" ~ "Crude oil price in US dollar",
      . == "price_huf" ~ "Crude oil price in HUF"
    ))
    ) %>% 
  na.omit() %>% 
  ggplot(aes(time, value, color = name)) + 
  geom_hline(yintercept = 1, lty = 2) +
  geom_line() + 
  scale_x_date(breaks = scales::date_breaks("months"), labels = ~ NiceMonth(., keep_year = TRUE, label = TRUE)) + 
  scale_y_continuous(labels = scales::percent) + 
  labs(x = NULL, y = "2021 jan 4 = 100%", color = NULL)

p2
```

<img src="figures/crude_price_index-1.png" style="display: block; margin: auto;" />

``` r
patchwork::wrap_plots(p1, p2, ncol = 1, heights = c(3, 1))
```

<img src="figures/crude_price_combined-1.png" style="display: block; margin: auto;" />

``` r
download_days <- fuel_df %>% 
  pull(date) %>% 
  unique() %>% 
  sort()

Sys.setlocale("LC_TIME", "C") # mac os specific language setup
```

    ## [1] "C"

``` r
calendR::calendR(
  start_date = "2021-08-01",
  end_date =  "2022-03-31",
  special.col = "lightblue",
  special.days = download_days - as.Date("2021-07-31"),
  start = "M"
)
```

<img src="figures/calendar-1.png" style="display: block; margin: auto;" />

## EDA
