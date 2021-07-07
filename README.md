# dp-embeds

1. Clone this repo and run `npm i && npm start`
2. Run the `dp_script.py` below to generate reports for the embeds:

## dp_script.py

```python
import altair as alt  # noqa
import matplotlib.pyplot as plt  # noqa
import plotly.graph_objects as go  # noqa
from bokeh.plotting import figure  # noqa
import folium  # noqa
import pandas as pd
from vega_datasets import data as vega_data
import random
import datapane as dp
import numpy as np
import plotly.express as px
from bokeh.sampledata.iris import flowers


def gen_table_df(rows: int = 4, alphabet: str = "ABCDEF") -> pd.DataFrame:
    data = [{x: random.randint(0, 1000) for x in alphabet} for _ in range(0, rows)]
    return pd.DataFrame.from_dict(data)


def gen_df(dim: int = 4) -> pd.DataFrame:
    axis = [i for i in range(0, dim)]
    data = {"x": axis, "y": axis}
    return pd.DataFrame.from_dict(data)


def gen_bokeh(responsive=True, **kw):
    # Create scatter plot with Bokeh
    colormap = {'setosa': 'red', 'versicolor': 'green', 'virginica': 'blue'}
    colors = [colormap[x] for x in flowers['species']]

    bokeh_chart = figure(title="Iris Morphology", width=1500, height=1500, **kw)
    bokeh_chart.xaxis.axis_label = 'Petal Length'
    bokeh_chart.yaxis.axis_label = 'Petal Width'

    bokeh_chart.circle(flowers["petal_length"], flowers["petal_width"],
                       color=colors, fill_alpha=0.2, size=10)

    # Publish the report
    return dp.Plot(bokeh_chart, responsive=responsive)


def gen_plotly(responsive=True, **kw):
    fig = go.Figure()
    fig.add_trace(go.Scatter(x=[0, 1, 2, 3, 4, 5], y=[1.5, 1, 1.3, 0.7, 0.8, 0.9]))
    fig.add_trace(go.Bar(x=[0, 1, 2, 3, 4, 5], y=[1, 0.5, 0.7, -1.2, 0.3, 0.4]))
    fig.update_layout(**kw)
    return dp.Plot(fig, responsive=responsive)


def gen_plotly_express(responsive=True, **kw):
    df = px.data.gapminder()
    plotly_chart = px.scatter(
        df.query("year==2007"), x="gdpPercap", y="lifeExp",
        size="pop", color="continent",
        hover_name="country", log_x=True, size_max=60,
        width=1500, height=1500,
        **kw
    )
    return dp.Plot(plotly_chart, responsive=responsive)


def gen_vega(responsive=True, **kw):
    return dp.Plot(
        data=alt.Chart(gen_df()).properties(width=1500, height=1500, **kw).mark_line().encode(x="x", y="y"),
        responsive=responsive
    )


def gen_vega_bindings():
    gap = pd.read_json(vega_data.gapminder.url)
    select_year = alt.selection_single(
        name='select', fields=['year'], init={'year': 1955},
        bind=alt.binding_range(min=1955, max=2005, step=5)
    )
    alt_chart = alt.Chart(gap).mark_point(filled=True).encode(
        alt.X('fertility', scale=alt.Scale(zero=False)),
        alt.Y('life_expect', scale=alt.Scale(zero=False)),
        alt.Size('pop:Q'),
        alt.Color('cluster:N'),
        alt.Order('pop:Q', sort='descending'),
    ).add_selection(select_year).transform_filter(select_year)
    return alt_chart


def gen_matplotlib(responsive=True):
    gap = pd.read_json(vega_data.gapminder.url)
    fig = gap.plot.scatter(x='life_expect', y='fertility')
    return dp.Plot(fig, responsive=responsive)


def gen_folium():
    return dp.Plot(folium.Map(location=[45.372, -121.6972], zoom_start=12, tiles="Stamen Terrain"))


def gen_plotly_bindings(responsive=True):
    # Create figure
    fig = go.Figure()

    # Add traces, one for each slider step
    for step in np.arange(0, 5, 0.1):
        fig.add_trace(
            go.Scatter(
                visible=False,
                line=dict(color="#00CED1", width=6),
                name="𝜈 = " + str(step),
                x=np.arange(0, 10, 0.01),
                y=np.sin(step * np.arange(0, 10, 0.01))))

    # Make 10th trace visible
    fig.data[10].visible = True

    # Create and add slider
    steps = []
    for i in range(len(fig.data)):
        step = dict(
            method="update",
            args=[{"visible": [False] * len(fig.data)},
                  {"title": "Slider switched to step: " + str(i)}],  # layout attribute
        )
        step["args"][0]["visible"][i] = True  # Toggle i'th trace to "visible"
        steps.append(step)

    sliders = [dict(
        active=10,
        currentvalue={"prefix": "Frequency: "},
        pad={"t": 50},
        steps=steps
    )]

    fig.update_layout(
        sliders=sliders
    )
    return dp.Plot(fig, responsive=responsive)

dp.Report(gen_vega(responsive=False)).publish(name="vega-report-unresponsive")
dp.Report(gen_vega()).publish(name="vega-report-responsive")
dp.Report(gen_vega_bindings()).publish(name="vega-report-bindings")
dp.Report(gen_plotly_express(responsive=False)).publish(name="plotly-report-unresponsive")
dp.Report(gen_plotly_express()).publish(name="plotly-report-responsive")
dp.Report(gen_plotly_bindings()).publish(name="plotly-report-bindings")
dp.Report(gen_matplotlib()).publish(name="mpl-report-responsive")
dp.Report(gen_matplotlib(responsive=False)).publish(name="mpl-report-unresponsive")
dp.Report(gen_bokeh(responsive=False)).publish(name="bokeh-report-unresponsive")
dp.Report(gen_bokeh()).publish(name="bokeh-report-responsive")
dp.Report(gen_table_df(rows=40)).publish(name="table-report")
dp.Report(dp.DataTable(gen_df(1000))).publish(name="datatable-report")
dp.Report(gen_folium()).publish(name="folium-report")
dp.Report(gen_vega_bindings(), gen_bokeh(), gen_plotly_express(), gen_matplotlib(), gen_folium()).publish(name="multi-report-responsive")
```
