<a href="https://www.kaggle.com/code/jesusgraterol/utilities" target="_blank">
  <img align="left" alt="Kaggle" title="Open in Kaggle" src="https://kaggle.com/static/images/open-in-kaggle.svg">
</a><br><br>

# Notebook Utilities

The purpose of this Notebook/Utility Script is to provide a series of helper functions that will simplify the development of more complex projects.


```python
##################
## Dependencies ##
##################

!pip install mplfinance
from typing import Union, Literal, Dict, List, Tuple
from os import walk
from os.path import join
from json import load
from datetime import datetime
from pandas import DataFrame, to_datetime
import mplfinance as mpl 
```

## Number Helpers

```python
def calculate_percentage_change(old_value: float, new_value: float, precision: int = 2) -> float:
    """Calculates the change in % a number has experienced.
    
    Args:
        old_value: float
            The original value prior to experiencing any changes.
        new_value: float
            The new value after the changes took place.
        precision: int
            The number of decimals that will be used to round the result.
            
    Returns:
        float
    """
    # If the original value is 0, it is impossible to calculate the % change
    if old_value == 0:
        return 0
    
    # Init the change
    change: float = 0.0
        
    # Handle the case in which the value experienced an increase
    if new_value > old_value:
        increase: float = new_value - old_value
        change = (increase / old_value) * 100
        
    # Handle the case in which the value experienced a decrease
    elif old_value > new_value:
        decrease: float = old_value - new_value
        change = -((decrease / old_value) * 100)
        
    # Finally, return the % change. Note that a value cannot experience a decrease larger than 100%
    return round(change if change >=-100 else -100, precision)
```

```python
def calculate_percentage_change(old_value: float, new_value: float, precision: int = 2) -> float:
    """Calculates the change in % a number has experienced.
    
    Args:
        old_value: float
            The original value prior to experiencing any changes.
        new_value: float
            The new value after the changes took place.
        precision: int
            The number of decimals that will be used to round the result.
            
    Returns:
        float
    """
    # If the original value is 0, it is impossible to calculate the % change
    if old_value == 0:
        return 0
    
    # Init the change
    change: float = 0.0
        
    # Handle the case in which the value experienced an increase
    if new_value > old_value:
        increase: float = new_value - old_value
        change = (increase / old_value) * 100
        
    # Handle the case in which the value experienced a decrease
    elif old_value > new_value:
        decrease: float = old_value - new_value
        change = -((decrease / old_value) * 100)
        
    # Finally, return the % change. Note that a value cannot experience a decrease larger than 100%
    return round(change if change >=-100 else -100, precision)
```

```python
def alter_number_by_percentage(value: float, percent: float, precision: int = 2) -> float:
    """Alters a number based on given percentage. For example, if value is 100 and percent
    is 50 it will return 150. On the other hand, if value is 100 and percent is -50 it will
    return 50.

    Args:
        value: float
            The number that will be altered.
        percent: float 
            The percentage that will be applied to the value.
        precision: int
            The number of decimals that will be used to round the result.

    Returns:
        float
    """
    # Init the new value
    new_value = value

    # Handle an increase
    if percent > 0:
        new_value = ((percent / 100) + 1) * value

    # Handle a decrease
    elif percent < 0:
        new_value = -(((percent * -1) / 100) - 1) * value

    # Return the altered number
    return round(new_value, precision)
```

## Time Helpers

```python
def from_date_string_to_timestamp(date_str: str) -> int:
    """Converts a Date String (DD/MM/YYYY) into unix time in milliseconds.
    
    Args:
        date_str: str
            The date string to be converted.
            
    Returns:
        int
    """
    if isinstance(date_str, str):
      date_split: List[str] = date_str.split('/')
      dt: datetime = datetime(int(date_split[2]), int(date_split[1]), int(date_split[0]))
      return int(dt.timestamp()*1000)
    else:
      return date_str
```

```python
def from_milliseconds_to_seconds(milliseconds: Union[int, float]) -> int:
    """Converts a time in milliseconds into seconds.
    
    Args:
        milliseconds: Union[int, float]
            The time in ms that will be converted.
    
    Returns: int
    """
    return int(milliseconds / 1000)
```

```python
def from_seconds_to_milliseconds(seconds: Union[int, float]) -> int:
    """Converts a unix timestamp based in seconds to milliseconds.
    
    Args:
        seconds: Union[int, float]
            The time in seconds that will be converted.
            
    Returns:
        int
    """
    return int(seconds * 1000)
```

```python
def add_minutes(timestamp_ms: Union[int, float], minutes: int) -> int:
    """Adds any number of minutes to a unix timestamp in milliseconds.
    
    Args:
        timestamp_ms: Union[int, float]
            The timestamp that will be altered.
        minutes: int
            The number of minutes that will be added to the timestamp.
            
            
    """
    return int(timestamp_ms + (from_seconds_to_milliseconds(60) * minutes))
```

## Output Prettifiers

```python
def currency(value: Union[int, float], symbol: str = "$") -> str:
    """Converts a money value into a readable string with any symbol
    as suffix.
    
    Args:
        value: Union[int, float]
            The number that will be prettified
        symbol: str
            The character that will be appended
    """
    return f"{value:,}{symbol}"
```

```python
def from_milliseconds_to_date_string(ms: int) -> str:
    """Converts a unix timestamp in milliseconds into a readeable date string.
    
    Args:
        ms: int
            The current time in milliseconds.
            
    Returns:
        str
    """
    return datetime.fromtimestamp(from_milliseconds_to_seconds(ms)).strftime("%d/%m/%Y, %H:%M:%S")
```

## Bitcoin Historic Candlesticks Retriever

```python
def get_historic_candlesticks(
        interval: IIntervalID, 
        start: Union[int, str, None] = None, 
        end: Union[int, str, None] = None
) -> DataFrame:
    """Loads the full candlesticks dataset and returns it for a given interval
    and date range (if any).
    
    Args:
        interval: IIntervalID
            The interval identifier of the desired dataset.
        start: Union[int, str, None]
            The date at which the historic candlesticks will begin. If None is provided,
            the ds will include all candlesticks from the very beginning.
        end: Union[int, str, None]
            The date at which the historic candlesticks will end. If None is provided,
            the ds will include all the candlesticks until the very end.
    
    * IMPORTANT: The accepted data types for the date range are:
        - A unix timestamp in milliseconds
        - A date string in the following format: DD/MM/YYYY
    
    Returns:
        DataFrame
    """
    # Load the entire dataset into RAM
    ds: IFullDataset = {}
    for dirname, _, filenames in walk('/kaggle/input'):
        for filename in filenames:
            if filename == "dataset.json":
                with open(join(dirname, filename), "r") as file_instance:
                    ds = load(file_instance)
                    
    # If a date range was not provided, return the entire ds
    if start is None and end is None:
        return DataFrame(ds[interval])
    
    # Otherwise, put together the date range based on the input
    else:
        # Initialize the dataframe
        df: DataFrame = DataFrame(ds[interval])
        
        # Initialize the real start and end timestamps based on its format
        real_start: Union[int, str, None] = from_date_string_to_timestamp(start) if isinstance(start, str) else start
        real_end: Union[int, str, None] = from_date_string_to_timestamp(end) if isinstance(end, str) else end
        
        # Handle the case where both values were provided
        if isinstance(real_start, (int, float)) and isinstance(real_end, (int, float)):
            df = df[(df["ot"] >= real_start) & (df["ot"] <= real_end)]
            
        # Handle the case where only the start was provided
        elif isinstance(real_start, (int, float)) and real_end is None:
            df = df[(df["ot"] >= real_start)]
            
        # Handle the case where only the end was provided
        else:
            df = df[(df["ot"] <= real_end)]
        
        # Reset the index so the returned df is fresh
        df.reset_index(drop=True, inplace=True)
            
        # Finally, return the final df
        return df
```

## Chart Plotting

```python
def plot_candlesticks(
    candlesticks: Union[List[dict], DataFrame], 
    title: Union[str, None] = None,
    figsize: Tuple[int, int] = (8, 5),
    display_volume: bool = True
) -> None:
    """Plots a candlestick chart based on provided params. If the number of records is 
    greater than 50k, it will plot a line chart instead.
    
    Args:
        candlesticks: Union[List[dict], DataFrame]
            The candlestick records in df or dict format.
        title: Union[str, None] = None
            The title that will be attached to the chart. 
        figsize: Tuple[int, int] = (8, 6)
            The size of the chart's figure that will be plotted.
        display_volume: bool
            If enabled, the volume will be displayed at the bottom of the chart.
    """
    # Init the df
    candlesticks_df = candlesticks.copy() if isinstance(candlesticks, DataFrame) else DataFrame(candlesticks)

    # Rename the columns to match the requirements
    if display_volume:
        candlesticks_df.rename(columns={"o": "Open", "h":"High", "l":"Low", "c": "Close", "v": "Volume"}, inplace=True)
    else:
        candlesticks_df.rename(columns={"o": "Open", "h":"High", "l":"Low", "c": "Close"}, inplace=True)

    # Set the period start as the indexes
    candlesticks_df["time_period_start"] = to_datetime(candlesticks_df.ot, unit="ms")
    candlesticks_df = candlesticks_df.set_index("time_period_start")

    # Plot the chart
    mpl.plot(
        candlesticks_df,
        type="candle" if candlesticks_df.shape[0] < 50000 else "line", 
        figsize=figsize,
        title=title if isinstance(title, str) else "",
        volume=display_volume,
        style="yahoo",
        warn_too_much_data=1000000,
    ) 
```