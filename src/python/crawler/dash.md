# Dash基础

本章介绍 Dash 的基本知识, 来自于官网教程

## 一. Dash Layout

Dash apps 由两部分组成，第一部分是 'Layout'，决定了 app 的外观. 第二部分描述了 app 如何进行交互, 这在下一节进行介绍.

```python
# Run this app with `python app.py` and
# visit http://127.0.0.1:8050/ in your web browser.

from dash import Dash, html, dcc
import plotly.express as px
import pandas as pd

app = Dash(__name__)

# assume you have a "long-form" data frame
# see https://plotly.com/python/px-arguments/ for more options
df = pd.DataFrame({
    "Fruit": ["Apples", "Oranges", "Bananas", "Apples", "Oranges", "Bananas"],
    "Amount": [4, 1, 2, 2, 4, 5],
    "City": ["SF", "SF", "SF", "Montreal", "Montreal", "Montreal"]
})

fig = px.bar(df, x="Fruit", y="Amount", color="City", barmode="group")

app.layout = html.Div(children=[
    html.H1(children='Hello Dash'),

    html.Div(children='''
        Dash: A web application framework for your data.
    '''),

    dcc.Graph(
        id='example-graph',
        figure=fig
    )
])

if __name__ == '__main__':
    app.run_server(debug=True)

```

- Dash HTML 模组 `dash.html` 对每个 HTML 标签都有对应的元件，`html.H1(children='Hello Dash')`为你的 app 生成了一个`<h1>Hello Dash</h1>` html标签.
- 不是所有元件都是纯的 html. Dash Core Components 模组(`dash.dcc`)包含了交互式高级元件, 它们是由 React.js 库通过 JavaScript, HTML, and CSS 生成的.
- 每个元件完全由关键字属性进行定义
- `children` 是特殊的, 按照惯例，它永远是第一个属性，也就是说可以省略它，`html.H1(children='Hello Dash')` 和 `html.H1('Hello Dash')` 是一样的. 它可以是一个字符串, 一个数字, 一个单独的元件或者一个元件列表.
- 上述代码自己运行的 app 和官网上的字体看起来有所不一样, 因为官网上使用了本地的 CSS 风格. [具体使用 CSS 的教程](https://dash.plotly.com/external-resources)

### 1.1 热重载

Dash支持热重载(hot-reloading), 这一特性可以通过设定`app.run_server(debug=True)`进行启用.

### 1.2 HTML 元件

Dash HTML Components(`dash.html`)对应每个HTML标签包含了一个元件类，以及对应每个HTML变量同样有关键词变量.

对上一节的代码进行修改

```python
# Run this app with `python app.py` and
# visit http://127.0.0.1:8050/ in your web browser.

from dash import Dash, dcc, html
import plotly.express as px
import pandas as pd

app = Dash(__name__)

colors = {
    'background': '#111111',
    'text': '#7FDBFF'
}

# assume you have a "long-form" data frame
# see https://plotly.com/python/px-arguments/ for more options
df = pd.DataFrame({
    "Fruit": ["Apples", "Oranges", "Bananas", "Apples", "Oranges", "Bananas"],
    "Amount": [4, 1, 2, 2, 4, 5],
    "City": ["SF", "SF", "SF", "Montreal", "Montreal", "Montreal"]
})

fig = px.bar(df, x="Fruit", y="Amount", color="City", barmode="group")

fig.update_layout(
    plot_bgcolor=colors['background'],
    paper_bgcolor=colors['background'],
    font_color=colors['text']
)

app.layout = html.Div(style={'backgroundColor': colors['background']}, children=[
    html.H1(
        children='Hello Dash',
        style={
            'textAlign': 'center',
            'color': colors['text']
        }
    ),

    html.Div(children='Dash: A web application framework for your data.', style={
        'textAlign': 'center',
        'color': colors['text']
    }),

    dcc.Graph(
        id='example-graph-2',
        figure=fig
    )
])

if __name__ == '__main__':
    app.run_server(debug=True)

```

本例中, 我们修改了通过`style` 属性修改了 `html.Div`和`html.H1components`的行内风格.

```python
html.H1('Hello Dash', style={'textAlign': 'center', 'color': '#7FDBFF'})
```
上述代码在Dash app中被渲染为`<h1 style="text-align: center; color: #7FDBFF">Hello Dash</h1>`.

在 `dash.html` 和 HTML 属性间有几个重要不同
- HTML 中的 `style` 是分号间隔的字符串, 而在Dash中, 可以仅写为一个字典
- `style`字典中的键是**驼峰式(camelCased)**的, 所以 `text-align` 应该是 `textAlign`
- HEML `class` 属性在 Dash 中是类名
- HTML标签中的 children 通过关键词变量 `children` 进行指定, 按照惯例这是第一个变量通常可以省略

### 1.3 可再用的元件

下面是一个例子，从 Pandas 的数据帧中生成一个 Table

```python
# Run this app with `python app.py` and
# visit http://127.0.0.1:8050/ in your web browser.

from dash import Dash, html
import pandas as pd

df = pd.read_csv('https://gist.githubusercontent.com/chriddyp/c78bf172206ce24f77d6363a2d754b59/raw/c353e8ef842413cae56ae3920b8fd78468aa4cb2/usa-agricultural-exports-2011.csv')


def generate_table(dataframe, max_rows=10):
    return html.Table([
        html.Thead(
            html.Tr([html.Th(col) for col in dataframe.columns])
        ),
        html.Tbody([
            html.Tr([
                html.Td(dataframe.iloc[i][col]) for col in dataframe.columns
            ]) for i in range(min(len(dataframe), max_rows))
        ])
    ])


app = Dash(__name__)

app.layout = html.Div([
    html.H4(children='US Agriculture Exports (2011)'),
    generate_table(df)
])

if __name__ == '__main__':
    app.run_server(debug=True)

```

### 1.4 更多可视化

Dash Core Components(`dash.dcc`) 包含了一个叫做 `Graph` 的元件.

`Graph` 通过开源的 plotly.js 图形库渲染出交互式可视化数据. Plotly.js 支持超过35类图形，并且可以渲染为矢量 SVG 或者 WebGL.

`Graph` 元件中使用的 `figure` 变量和开源 python 图形库 `plotlt.py` 中使用的 `figure` 是一样的.

```python
# Run this app with `python app.py` and
# visit http://127.0.0.1:8050/ in your web browser.

from dash import Dash, dcc, html
import plotly.express as px
import pandas as pd


app = Dash(__name__)

df = pd.read_csv('https://gist.githubusercontent.com/chriddyp/5d1ea79569ed194d432e56108a04d188/raw/a9f9e8076b837d541398e999dcbac2b2826a81f8/gdp-life-exp-2007.csv')

fig = px.scatter(df, x="gdp per capita", y="life expectancy",
                 size="population", color="continent", hover_name="country",
                 log_x=True, size_max=60)

app.layout = html.Div([
    dcc.Graph(
        id='life-exp-vs-gdp',
        figure=fig
    )
])

if __name__ == '__main__':
    app.run_server(debug=True)

```

### 1.5 Markdown

可以使用 Dash Core Components(`dash.dcc`) 中的 Markdown 元件来写

```python
# Run this app with `python app.py` and
# visit http://127.0.0.1:8050/ in your web browser.

from dash import Dash, html, dcc

app = Dash(__name__)

markdown_text = '''
### Dash and Markdown

Dash apps can be written in Markdown.
Dash uses the [CommonMark](http://commonmark.org/)
specification of Markdown.
Check out their [60 Second Markdown Tutorial](http://commonmark.org/help/)
if this is your first introduction to Markdown!
'''

app.layout = html.Div([
    dcc.Markdown(children=markdown_text)
])

if __name__ == '__main__':
    app.run_server(debug=True)

```

### 1.6 核心元件

可以在[Dash Core Components](https://dash.plotly.com/dash-core-components)中查看所有的核心元件介绍

```python
# Run this app with `python app.py` and
# visit http://127.0.0.1:8050/ in your web browser.

from dash import Dash, html, dcc

app = Dash(__name__)

app.layout = html.Div([
    html.Div(children=[
        html.Label('Dropdown'),
        dcc.Dropdown(['New York City', 'Montréal', 'San Francisco'], 'Montréal'),

        html.Br(),
        html.Label('Multi-Select Dropdown'),
        dcc.Dropdown(['New York City', 'Montréal', 'San Francisco'],
                     ['Montréal', 'San Francisco'],
                     multi=True),

        html.Br(),
        html.Label('Radio Items'),
        dcc.RadioItems(['New York City', 'Montréal', 'San Francisco'], 'Montréal'),
    ], style={'padding': 10, 'flex': 1}),

    html.Div(children=[
        html.Label('Checkboxes'),
        dcc.Checklist(['New York City', 'Montréal', 'San Francisco'],
                      ['Montréal', 'San Francisco']
        ),

        html.Br(),
        html.Label('Text Input'),
        dcc.Input(value='MTL', type='text'),

        html.Br(),
        html.Label('Slider'),
        dcc.Slider(
            min=0,
            max=9,
            marks={i: f'Label {i}' if i == 1 else str(i) for i in range(1, 6)},
            value=5,
        ),
    ], style={'padding': 10, 'flex': 1})
], style={'display': 'flex', 'flex-direction': 'row'})

if __name__ == '__main__':
    app.run_server(debug=True)

```

## 二. 基本的Dash回调

本节将介绍如何使用回调函数(callback functions)来制作 Dash app, 当输入元件的属性改变时，Dash将自动调用回调函数，(作为输出)来更新另一个元件的某些属性.

### 2.1 简单的交互式 Dash app

```python
from dash import Dash, dcc, html, Input, Output

app = Dash(__name__)

app.layout = html.Div([
    html.H6("Change the value in the text box to see callbacks in action!"),
    html.Div([
        "Input: ",
        dcc.Input(id='my-input', value='initial value', type='text')
    ]),
    html.Br(),
    html.Div(id='my-output'),

])


@app.callback(
    Output(component_id='my-output', component_property='children'),
    Input(component_id='my-input', component_property='value')
)
def update_output_div(input_value):
    return f'Output: {input_value}'


if __name__ == '__main__':
    app.run_server(debug=True)

```

- 'inputs' 和 'outputs' 在应用中描述为装饰器 `@app.callback` 的变量
- 在Dash中，应用的 input 和 output 仅仅是特定元件的属性. 在上例中，input是元件的`'value'`属性, 它有 ID `'my-input'`，output是元件的`'children'`属性，它有ID `'my-output'`.
- 只要一个 input 属性改变了，回调装饰器所包装的函数就会自动被调用. Dash 将新的 input 输入进这个回调函数中，并且将函数输出更新为 output 元件
- `component_id` 和 `component_property` 关键字是可选的
- 不要混淆 `dash.dependencies.Input` 和 `dcc.Input`，前者只用在回调定义中，后者是是实际的元件
- 注意到我们在`layout`的`my-output`元件中没有设置`children`属性. 当Dash app启动时，它会用输入元件的初始值来自动调用所有的回调函数，从而决定输出元件的初始状态. 在本例中，如果通过 `html.Div(id='my-output', children='Hello world')` 指定了 div 元件，在 app 启动时它会被覆盖

回忆每个元件是如何通过一组关键字变量来进行完全描述的，这些在Python中设定的变量成为了元件的属性. 通过 Dash 的交互式，我们可以使用回调函数来动态地更新这些属性，通常我们会更新 HTML 元件的`children`属性来展示新的文本，或者 `dcc.Graph` 元件的 `figure` 属性来展示新的数据，我们也可以更新一个元件的 `style` 甚至是一个 `dcc.Dropdown` 元件的可用'options'!

下述例子中 `dcc.Slider` 更新 `dcc.Graph`

```python
from dash import Dash, dcc, html, Input, Output
import plotly.express as px

import pandas as pd

df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminderDataFiveYear.csv')

app = Dash(__name__)

app.layout = html.Div([
    dcc.Graph(id='graph-with-slider'),
    dcc.Slider(
        df['year'].min(),
        df['year'].max(),
        step=None,
        value=df['year'].min(),
        marks={str(year): str(year) for year in df['year'].unique()},
        id='year-slider'
    )
])


@app.callback(
    Output('graph-with-slider', 'figure'),
    Input('year-slider', 'value'))
def update_figure(selected_year):
    filtered_df = df[df.year == selected_year]

    fig = px.scatter(filtered_df, x="gdpPercap", y="lifeExp",
                     size="pop", color="continent", hover_name="country",
                     log_x=True, size_max=55)

    fig.update_layout(transition_duration=500)

    return fig


if __name__ == '__main__':
    app.run_server(debug=True)

```

### 2.2 多个输出的 Dash app

下面的例子中，一个回调函数绑定了5个输入，分别是两个 `dcc.Dropdown` 元件，两个 `dcc.RadioItems` 元件以及一个 `dcc.Slider` 元件的 `value` 属性，回调输出为一个 `dcc.Graph` 元件的 `figure` 属性.

```python
from dash import Dash, dcc, html, Input, Output
import plotly.express as px

import pandas as pd

app = Dash(__name__)

df = pd.read_csv('https://plotly.github.io/datasets/country_indicators.csv')

app.layout = html.Div([
    html.Div([

        html.Div([
            dcc.Dropdown(
                df['Indicator Name'].unique(),
                'Fertility rate, total (births per woman)',
                id='xaxis-column'
            ),
            dcc.RadioItems(
                ['Linear', 'Log'],
                'Linear',
                id='xaxis-type',
                inline=True
            )
        ], style={'width': '48%', 'display': 'inline-block'}),

        html.Div([
            dcc.Dropdown(
                df['Indicator Name'].unique(),
                'Life expectancy at birth, total (years)',
                id='yaxis-column'
            ),
            dcc.RadioItems(
                ['Linear', 'Log'],
                'Linear',
                id='yaxis-type',
                inline=True
            )
        ], style={'width': '48%', 'float': 'right', 'display': 'inline-block'})
    ]),

    dcc.Graph(id='indicator-graphic'),

    dcc.Slider(
        df['Year'].min(),
        df['Year'].max(),
        step=None,
        id='year--slider',
        value=df['Year'].max(),
        marks={str(year): str(year) for year in df['Year'].unique()},

    )
])


@app.callback(
    Output('indicator-graphic', 'figure'),
    Input('xaxis-column', 'value'),
    Input('yaxis-column', 'value'),
    Input('xaxis-type', 'value'),
    Input('yaxis-type', 'value'),
    Input('year--slider', 'value'))
def update_graph(xaxis_column_name, yaxis_column_name,
                 xaxis_type, yaxis_type,
                 year_value):
    dff = df[df['Year'] == year_value]

    fig = px.scatter(x=dff[dff['Indicator Name'] == xaxis_column_name]['Value'],
                     y=dff[dff['Indicator Name'] == yaxis_column_name]['Value'],
                     hover_name=dff[dff['Indicator Name'] == yaxis_column_name]['Country Name'])

    fig.update_layout(margin={'l': 40, 'b': 40, 't': 10, 'r': 0}, hovermode='closest')

    fig.update_xaxes(title=xaxis_column_name,
                     type='linear' if xaxis_type == 'Linear' else 'log')

    fig.update_yaxes(title=yaxis_column_name,
                     type='linear' if yaxis_type == 'Linear' else 'log')

    return fig


if __name__ == '__main__':
    app.run_server(debug=True)

```

### 2.3 多输出的Dassh app

```python
from dash import Dash, dcc, html
from dash.dependencies import Input, Output

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = Dash(__name__, external_stylesheets=external_stylesheets)

app.layout = html.Div([
    dcc.Input(
        id='num-multi',
        type='number',
        value=5
    ),
    html.Table([
        html.Tr([html.Td(['x', html.Sup(2)]), html.Td(id='square')]),
        html.Tr([html.Td(['x', html.Sup(3)]), html.Td(id='cube')]),
        html.Tr([html.Td([2, html.Sup('x')]), html.Td(id='twos')]),
        html.Tr([html.Td([3, html.Sup('x')]), html.Td(id='threes')]),
        html.Tr([html.Td(['x', html.Sup('x')]), html.Td(id='x^x')]),
    ]),
])


@app.callback(
    Output('square', 'children'),
    Output('cube', 'children'),
    Output('twos', 'children'),
    Output('threes', 'children'),
    Output('x^x', 'children'),
    Input('num-multi', 'value'))
def callback_a(x):
    return x**2, x**3, 2**x, 3**x, x**x


if __name__ == '__main__':
    app.run_server(debug=True)

```

### 2.4 链式回调(chained callbacks)的Dash app

一个回调函数的输出可以是另一个回调函数的输入，这可以用来创建动态的UI.
下面的例子中，一个输入元件会更新另一个元件的可用选项.

```python
# -*- coding: utf-8 -*-
from dash import Dash, dcc, html, Input, Output

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = Dash(__name__, external_stylesheets=external_stylesheets)

all_options = {
    'America': ['New York City', 'San Francisco', 'Cincinnati'],
    'Canada': [u'Montréal', 'Toronto', 'Ottawa']
}
app.layout = html.Div([
    dcc.RadioItems(
        list(all_options.keys()),
        'America',
        id='countries-radio',
    ),

    html.Hr(),

    dcc.RadioItems(id='cities-radio'),

    html.Hr(),

    html.Div(id='display-selected-values')
])


@app.callback(
    Output('cities-radio', 'options'),
    Input('countries-radio', 'value'))
def set_cities_options(selected_country):
    return [{'label': i, 'value': i} for i in all_options[selected_country]]


@app.callback(
    Output('cities-radio', 'value'),
    Input('cities-radio', 'options'))
def set_cities_value(available_options):
    return available_options[0]['value']


@app.callback(
    Output('display-selected-values', 'children'),
    Input('countries-radio', 'value'),
    Input('cities-radio', 'value'))
def set_display_children(selected_country, selected_city):
    return u'{} is a city in {}'.format(
        selected_city, selected_country,
    )


if __name__ == '__main__':
    app.run_server(debug=True)

```

### 2.5 带状态(state)的 Dash app

某些情况下，不是实时根据用户的输入进行更新，而是等待输入完了所有的字符后再进行更新.
`state` 允许用户在不调用回调函数的情况下传递一些值. 下面例子中两个 `dcc.Input` 元件作为 `state`，一个新的按钮元件作为 `Input`(大写I，这是一个元件类型名).

```python
# -*- coding: utf-8 -*-
from dash import Dash, dcc, html
from dash.dependencies import Input, Output, State

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = Dash(__name__, external_stylesheets=external_stylesheets)

app.layout = html.Div([
    dcc.Input(id='input-1-state', type='text', value='Montréal'),
    dcc.Input(id='input-2-state', type='text', value='Canada'),
    html.Button(id='submit-button-state', n_clicks=0, children='Submit'),
    html.Div(id='output-state')
])


@app.callback(Output('output-state', 'children'),
              Input('submit-button-state', 'n_clicks'),
              State('input-1-state', 'value'),
              State('input-2-state', 'value'))
def update_output(n_clicks, input1, input2):
    return u'''
        The Button has been pressed {} times,
        Input 1 is "{}",
        and Input 2 is "{}"
    '''.format(n_clicks, input1, input2)


if __name__ == '__main__':
    app.run_server(debug=True)

```

### 2.6 将元件传入回调函数，而不是ID

在第一个例子中，有`id`为'my-input'的`dcc.Input`元件和`id`为'my-output'的`html.Div`元件，

```python
app.layout = html.Div([
    html.H6("Change the value in the text box to see callbacks in action!"),
    html.Div([
        "Input: ",
        dcc.Input(id='my-input', value='initial value', type='text')
    ]),
    html.Br(),
    html.Div(id='my-output'),

@app.callback(
    Output(component_id='my-output', component_property='children'),
    Input(component_id='my-input', component_property='value')
)
def update_output_div(input_value):
    return f'Output: {input_value}'
```

也可以不带`id`，直接将元件作为输入或者输出，Dash为这些元件自动生成ID.
下面的例子就是第一个例子，在声明 app layout 之前，创建了两个元件并分别指派给一个变量，然后就可以在 layout 中直接引用这些变量并且直接将它们作为输入或者输出传给回调函数.

```python
from dash import Dash, dcc, html, Input, Output, callback

app = Dash(__name__)

my_input = dcc.Input(value='initial value', type='text')
my_output = html.Div()

app.layout = html.Div([
    html.H6("Change the value in the text box to see callbacks in action!"),
    html.Div([
        "Input: ",
        my_input
    ]),
    html.Br(),
    my_output
])


@callback(
    Output(my_output, component_property='children'),
    Input(my_input, component_property='value')
)
def update_output_div(input_value):
    return f'Output: {input_value}'


if __name__ == '__main__':
    app.run_server(debug=True)

```

## 三. 交互式可视化

`dcc.Graph` 元件有四个可以在交互中改变的属性: `hoverData`, `clickData`, `selectedData`, `relayoutData`.

```python
import json

from dash import Dash, dcc, html
from dash.dependencies import Input, Output
import plotly.express as px
import pandas as pd

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = Dash(__name__, external_stylesheets=external_stylesheets)

styles = {
    'pre': {
        'border': 'thin lightgrey solid',
        'overflowX': 'scroll'
    }
}

df = pd.DataFrame({
    "x": [1,2,1,2],
    "y": [1,2,3,4],
    "customdata": [1,2,3,4],
    "fruit": ["apple", "apple", "orange", "orange"]
})

fig = px.scatter(df, x="x", y="y", color="fruit", custom_data=["customdata"])

fig.update_layout(clickmode='event+select')

fig.update_traces(marker_size=20)

app.layout = html.Div([
    dcc.Graph(
        id='basic-interactions',
        figure=fig
    ),

    html.Div(className='row', children=[
        html.Div([
            dcc.Markdown("""
                **Hover Data**

                Mouse over values in the graph.
            """),
            html.Pre(id='hover-data', style=styles['pre'])
        ], className='three columns'),

        html.Div([
            dcc.Markdown("""
                **Click Data**

                Click on points in the graph.
            """),
            html.Pre(id='click-data', style=styles['pre']),
        ], className='three columns'),

        html.Div([
            dcc.Markdown("""
                **Selection Data**

                Choose the lasso or rectangle tool in the graph's menu
                bar and then select points in the graph.

                Note that if `layout.clickmode = 'event+select'`, selection data also
                accumulates (or un-accumulates) selected data if you hold down the shift
                button while clicking.
            """),
            html.Pre(id='selected-data', style=styles['pre']),
        ], className='three columns'),

        html.Div([
            dcc.Markdown("""
                **Zoom and Relayout Data**

                Click and drag on the graph to zoom or click on the zoom
                buttons in the graph's menu bar.
                Clicking on legend items will also fire
                this event.
            """),
            html.Pre(id='relayout-data', style=styles['pre']),
        ], className='three columns')
    ])
])


@app.callback(
    Output('hover-data', 'children'),
    Input('basic-interactions', 'hoverData'))
def display_hover_data(hoverData):
    return json.dumps(hoverData, indent=2)


@app.callback(
    Output('click-data', 'children'),
    Input('basic-interactions', 'clickData'))
def display_click_data(clickData):
    return json.dumps(clickData, indent=2)


@app.callback(
    Output('selected-data', 'children'),
    Input('basic-interactions', 'selectedData'))
def display_selected_data(selectedData):
    return json.dumps(selectedData, indent=2)


@app.callback(
    Output('relayout-data', 'children'),
    Input('basic-interactions', 'relayoutData'))
def display_relayout_data(relayoutData):
    return json.dumps(relayoutData, indent=2)


if __name__ == '__main__':
    app.run_server(debug=True)

```

## 四. 在回调函数之间共享数据

暂时用不到
> dash app 是设计在多用户的场景中的，一个用户修改全局变量会导致其他用户的崩溃，所以要用`dcc.store`来储存共享数据

## 五. 实战:如何在已有的plotly图中手动添加曲线, 并根据曲线上的点进行计算？

### 通过drawmode手动画线

[Interactive annotations and shape drawing in plotly figures and Dash apps](https://eoss-image-processing.github.io/2020/05/06/shape-drawing.html)

注意离线plot中无法正确显示模式按钮`modeBarButtonsToAdd`, 需要在终端中运行

```python
import plotly.express as px
from skimage import data
img = data.chelsea() # or any image represented as a numpy array
fig = px.imshow(img)
# Define dragmode, newshape parameters, amd add modebar buttons
fig.update_layout(
    dragmode='drawrect', # define dragmode
    newshape=dict(line_color='cyan'))
# Add modebar buttons
fig.show(config={'modeBarButtonsToAdd':['drawline',
                                        'drawopenpath',
                                        'drawclosedpath',
                                        'drawcircle',
                                        'drawrect',
                                        'eraseshape'
                                       ]})
```

也能够方便地与Dash进行互动

```python
import dash
from dash.dependencies import Input, Output, State
import dash_html_components as html
import dash_core_components as dcc
import plotly.express as px
from skimage import data
import math

app = dash.Dash(__name__)

img = data.coins() # or any image represented as a numpy array

fig = px.imshow(img, color_continuous_scale='gray')
fig.update_layout(dragmode='drawline', newshape_line_color='cyan')

app.layout = html.Div(children=[
        dcc.Graph(
            id='graph',
            figure=fig,
            config={'modeBarButtonsToAdd':['drawline']}),
        html.Pre(id='content', children='Length of lines (pixels) \n')
        ], style={'width':'25%'})


@app.callback(
    dash.dependencies.Output('content', 'children'),
    [dash.dependencies.Input('graph', 'relayoutData')],
    [dash.dependencies.State('content', 'children')])
def shape_added(fig_data, content):
    if fig_data is None:
        return dash.no_update
    if 'shapes' in fig_data:
        line = fig_data['shapes'][-1]
        length = math.sqrt((line['x1'] - line['x0']) ** 2 +
                           (line['y1'] - line['y0']) ** 2)
        content += '%.1f'%length + '\n'
    return content

if __name__ == '__main__':
    app.run_server(debug=True)
```