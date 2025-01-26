[Introduction](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#introduction)
-------------------------------------------------------------------------------------------------------------

Pine Script™ v6 introduces a number of changes and new features. See the [Release Notes](https://www.tradingview.com/pine-script-docs/release-notes/) for a list of all new features.

Some changes are not compatible with v5 scripts. This guide explains how to update your script from v5 to v6. If you want to convert a script from v4 or earlier to v6, refer to the migration guides for previous versions and update the script one version at a time.

The Pine Editor converter can handle many of these changes _automatically_, while other changes might require _manual_ fixes.

Here are the changes that affect v5 scripts:

*   Values of the “int” and “float” types are no longer implicitly cast to “bool”.
*   Boolean values can no longer be [na](https://www.tradingview.com/pine-script-reference/v6/#var_na), and the [na()](https://www.tradingview.com/pine-script-reference/v6/#fun_na), [nz()](https://www.tradingview.com/pine-script-reference/v6/#fun_nz), and [fixnan()](https://www.tradingview.com/pine-script-reference/v6/#fun_fixnan) functions no longer accept “bool” arguments.
*   The [and](https://www.tradingview.com/pine-script-reference/v6/#kw_and) and [or](https://www.tradingview.com/pine-script-reference/v6/#kw_or) operators now evaluate conditions lazily.
*   All `request.*()` functions can now execute dynamically.
*   Division of two “const int” values can now return a fractional value.
*   The `when` parameter is removed from all applicable `strategy.*()` functions.
*   The default long and short margin percentage for strategies is now 100.
*   Strategies now trim the oldest orders in their results instead of raising an error when they exceed the 9000 trade limit.
*   The [strategy.exit()](https://www.tradingview.com/pine-script-reference/v6/#fun_strategy.exit) command no longer ignores relative parameters defining take-profit and stop-loss prices or trailing stop activation levels when the call also includes arguments for the related absolute parameters.
*   The history-referencing operator [\[\]](https://www.tradingview.com/pine-script-reference/v6/#op_%5B%5D) can no longer reference the history of literal values or fields of user-defined types directly.
*   Function calls can no longer include more than one argument for the same parameter.
*   The `offset` parameter of [plot()](https://www.tradingview.com/pine-script-reference/v6/#fun_plot) and other functions no longer accepts “series” values.
*   [na](https://www.tradingview.com/pine-script-reference/v6/#var_na) values are no longer allowed in place of built-in constants of unique types.
*   The value of [timeframe.period](https://www.tradingview.com/pine-script-reference/v6/#var_timeframe.period) now always includes a multiplier (e.g., `"1D"` instead of `"D"`).
*   Some `array.*()` functions now accept negative index arguments.
*   Some mutable variables are no longer erroneously marked as “const”.
*   The `transp` parameter is removed from all applicable functions.
*   Some default colors and color constants have updated values.

[Converting v5 to v6 using the Pine Editor](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#converting-v5-to-v6-using-the-pine-editor)
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------

The Pine Editor can automatically convert a v5 script to v6. The Pine Editor highlights the `//@version=5` [annotation](https://www.tradingview.com/pine-script-reference/v5/#an_@version=) of a v5 script in yellow.

To convert the script, click the editor’s “Manage script” dropdown menu and select “Convert code to v6”:

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-Convert-button.BtTphLrQ_ZFYojM.webp)

A script can only be converted if its v5 code compiles successfully. In rare cases, converting the script automatically will result in a v6 script with compilation errors. In that case, the errors will be highlighted in the Editor, and they would have to be resolved by hand. Use the information in the following sections to convert the remaining v5 code manually.

[Dynamic requests](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#dynamic-requests)
---------------------------------------------------------------------------------------------------------------------

Scripts can call all `request.*()` functions _dynamically by default_ in Pine v6.

When scripts call `request.*()` functions _non-dynamically_, the context for the request must be known when the script first runs, and must remain _unchanged_ (static) throughout the script’s execution. Therefore, `ticker`, `timeframe`, and other parameters that specify the context must be of a “[_simple_](https://www.tradingview.com/pine-script-reference/v5/#op_simple)” qualifier. This restriction also prevents scripts using `request.*()` functions in loops like [for](https://www.tradingview.com/pine-script-reference/v6/#kw_for) and [while](https://www.tradingview.com/pine-script-reference/v6/#kw_while).

In Pine v5, requests are by default not dynamic. Dynamic requests _are_ supported in v5, but only if programmers specify `dynamic_requests=true` in the script’s [indicator()](https://www.tradingview.com/pine-script-reference/v6/#fun_indicator), [strategy()](https://www.tradingview.com/pine-script-reference/v6/#fun_strategy), or [library()](https://www.tradingview.com/pine-script-reference/v5/#fun_library) declaration. Otherwise, the `dynamic_requests` parameter’s default value is `false` in v5.

In Pine v6, dynamic requests are available by default. The compiler analyzes whether the dynamic request mode is needed and turns it off if it is unnecessary, for performance reasons.

This change means that v6 `request.*()` calls innately support “[_series_](https://www.tradingview.com/pine-script-reference/v6/#type_series)” arguments. This qualifier change enables users to:

*   Request symbols dynamically, even if they are not known on the first execution of a script.
    
*   Use [arrays](https://www.tradingview.com/pine-script-reference/v6/#type_arrays) to store symbols and timeframes.
    
*   Call `request.*()` functions inside of [loops](https://www.tradingview.com/pine-script-docs/language/loops/).
    
*   Use `request.*()` calls in [exported library functions](https://www.tradingview.com/pine-script-docs/concepts/libraries/#library-functions).
    

The following example v6 script uses a single [request.security()](https://www.tradingview.com/pine-script-reference/v6/#fun_request.security) instance in a [loop](https://www.tradingview.com/pine-script-docs/language/loops/) to dynamically request data from multiple symbols stored in an [array](https://www.tradingview.com/pine-script-reference/v6/#type_array). Each loop iteration retrieves a symbol’s [close](https://www.tradingview.com/pine-script-reference/v6/#var_close) price from its respective “1D” chart. The script then calculates and plots the [average](https://www.tradingview.com/pine-script-reference/v6/#fun_array.avg) close price for the selected symbols to create a simple custom index. In Pine v5, the indicator’s `dynamic_requests` parameter must be set to `true` to run this code without triggering a compilation error:

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-Dynamic-requests.CU--JEtI_Z1wsCBX.webp)

``//@version=6   indicator("Dynamic `request` demo")   //For v5: must add `dynamic_requests=true` to `indicator()` for this code to work.      //@variable User-input toggle to display each symbol's `close` price on chart alongside average `close`.   bool showSymbols = input.bool(false, "Plot symbol closes")      //@variable Persistent array of "string" symbol ticker IDs to request for our custom index.   var array<string> symbols = array.from("NASDAQ:MSFT", "NASDAQ:AAPL", "NASDAQ:GOOGL", "NASDAQ:NVDA")      //@variable Array storing the `close` prices for the `symbols` on each bar.   array<float> symCloses = array.new<float>()      // Loop through `symbols` and request daily `close` prices.    for [i, sym] in symbols       float reqClose = request.security(sym, "1D", close)       symCloses.push(reqClose)      // Calculate and plot the average `close` for the `symbols` to create our custom index plot.   float avgClose = symCloses.avg()   plot(avgClose, "Avg close", avgClose >= avgClose[1] ? color.green : color.red, 3)       // Plot each symbol's `close` for reference if `showSymbols` is `true`.   plot(showSymbols ? symCloses.get(0) : na, "MSFT", color.blue)   plot(showSymbols ? symCloses.get(1) : na, "AAPL", color.navy)   plot(showSymbols ? symCloses.get(2) : na, "GOOGL", color.aqua)   plot(showSymbols ? symCloses.get(3) : na, "NVDA", color.teal)   ``

There are minor differences between dynamic and non-dynamic requests in some obscure cases, for example, when passing the result of one [request.security()](https://www.tradingview.com/pine-script-reference/v6/#fun_request.security) call as the `expr` for another call. As a result, in rare cases, a valid v5 script without `dynamic_requests=true` can behave differently when converted to v6, even if nothing related to requests was changed.

**Fix:** In Pine v6, the [indicator()](https://www.tradingview.com/pine-script-reference/v6/#fun_indicator), [strategy()](https://www.tradingview.com/pine-script-reference/v6/#fun_strategy), and [library()](https://www.tradingview.com/pine-script-reference/v6/#fun_library) functions all include a `dynamic_requests` parameter, which is set to `true` by default. If you find differences between the behavior of your `request.*()` call in v5 and v6, you can pass `dynamic_requests=false` to force the dynamic behavior off and replicate the previous v5 behavior.

[Types](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#types)
-----------------------------------------------------------------------------------------------

The following changes have been made to how Pine handles types.

### [Explicit “bool” casting](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#explicit-bool-casting)

In Pine v6, “int” and “float” values are no longer implicitly cast to “bool”.

In Pine v5, values of “int” and “float” types can be implicitly cast to “bool” when an expression or function requires a boolean value. In such cases, `na`, `0`, or `0.0` are considered `false`, and _any other value_ is considered `true`.

For example, take a look at this conditional expression:

`color expr = bar_index ? color.green : color.red   `

It assigns `color.red` to `expr` on the _first_ bar of the chart, because that bar has a `bar_index` of 0, and then assigns `color.green` on _every_ following bar, because any _non-zero_ value is `true`. The ternary operator [`?:`](https://www.tradingview.com/pine-script-reference/v6/#op_?:) expects a “bool” expression for its condition, but in v5 it can also accept a numeric value as its conditional expression, which it automatically converts (implicitly casts) to a “bool”.

In v6, scripts must _explicitly_ cast a numeric value to “bool” to use it where a “bool” type is required.

**Fix:** Wrap the numeric value with the [`bool()`](https://www.tradingview.com/pine-script-reference/v6/#fun_bool) function to cast it explicitly.

`color expr = bool(bar_index) ? color.green : color.red   `

### [Boolean values cannot be \`na\`](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#boolean-values-cannot-be-na)

In v6, “bool” values can no longer be `na`. Consequently, the [`na()`](https://www.tradingview.com/pine-script-reference/v6/#fun_na), [`nz()`](https://www.tradingview.com/pine-script-reference/v6/#fun_nz), and [`fixnan()`](https://www.tradingview.com/pine-script-reference/v6/#fun_fixnan) functions no longer accept “bool” types.

In v5, “[bool](https://www.tradingview.com/pine-script-reference/v6/#type_bool)” variables have _three_ possible values: they can be `true`, `false`, or `na`. The boolean `na` value behaves differently from both `true` and `false`:

*   When implicitly cast to “bool”, `na` is evaluated as `false`.
    
*   The boolean `na` value is **not** considered equal to `false` when compared using the `==` operator.
    
*   When the boolean `na` value is passed to the [na()](https://www.tradingview.com/pine-script-reference/v6/#fun_na) function, it returns `true`, whereas `na(true)` and `na(false)` both return `false`.
    

To manage the boolean `na` value, the [na()](https://www.tradingview.com/pine-script-reference/v6/#fun_na), [nz()](https://www.tradingview.com/pine-script-reference/v6/#fun_nz), and [fixnan()](https://www.tradingview.com/pine-script-reference/v6/#fun_fixnan) functions in v5 have overloads that accept “bool” type arguments. This third boolean state leads to occasional confusion in v5 scripts.

In v6, this is no longer the case: a “bool” must be _either_ `true` or `false`, with **no** third state. This means that in v6 scripts:

*   A variable declared as “bool” can **no longer** be assigned `na` as its default value.
    
*   In conditional expressions like [if](https://www.tradingview.com/pine-script-reference/v5/#kw_if) and [switch](https://www.tradingview.com/pine-script-reference/v6/#kw_switch), if the return type of the expression is “bool”, any _unspecified_ condition returns `false` instead of `na`.
    
*   Expressions that returned a boolean `na` value in v5 now return `false`. For example, using the history-referencing operator `[]` on the very first bar of the dataset to request a historical value of a “bool” variable returned `na` in v5, because no past bars exist, but in Pine v6 it returns `false`.
    
*   Functions that explicitly check whether a value is `na` – specifically, [na()](https://www.tradingview.com/pine-script-reference/v6/#fun_na), [nz()](https://www.tradingview.com/pine-script-reference/v6/#fun_nz), and [fixnan()](https://www.tradingview.com/pine-script-reference/v6/#fun_fixnan) – do **not** accept “bool” arguments in v6.
    

This example v5 script creates a simple [strategy](https://www.tradingview.com/pine-script-docs/concepts/strategies/) that switches between long and short positions when two moving averages cross. An [if](https://www.tradingview.com/pine-script-reference/v6/#kw_if)\-statement assigns `true` or `false` to a “bool” variable `isLong` to track the trade’s long or short direction, using the strategy’s positive (> 0) or negative (< 0) [position size](https://www.tradingview.com/pine-script-docs/concepts/strategies/#position-sizing). However, when the position size is zero, _neither_ of these conditions are valid. In v5, the undefined condition (== 0) assigns `na` to the variable `isLong`.

Therefore, a boolean `na` value occurs on the first few bars in the dataset before the strategy enters any positions. We can visualize the three “bool” states by setting the [background color](https://www.tradingview.com/pine-script-docs/concepts/backgrounds/) based on the value of `isLong`:

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-No-bool-na-1.B7b5WX8__ZhfW1O.webp)

``//@version=5   strategy("Bool `na` demo v5", overlay=true, margin_long=100, margin_short=100)      // Strategy's long and short trades are based on moving average cross over/under.   longCondition = ta.crossover(ta.sma(close, 14), ta.sma(close, 28))   if (longCondition)       strategy.entry("My Long Entry Id", strategy.long)      shortCondition = ta.crossunder(ta.sma(close, 14), ta.sma(close, 28))   if (shortCondition)       strategy.entry("My Short Entry Id", strategy.short)      //@variable Boolean variable that tracks the current direction of the trade.   // Is `true` when `position_size` is greater than 0 (long), and `false` when `position_size` is less than 0 (short).   bool isLong = if strategy.position_size > 0       true   else if strategy.position_size < 0       false   // When `position_size` is equal to 0, neither condition is met. In v5, an undefined condition sets `isLong` to `na`.      //@variable Background color, set depending on the state of `isLong` (`true`/`false`/`na`).   color stateColor = switch       isLong == true  => color.new(color.blue, 90)        // Blue color if long position.       isLong == false => color.new(color.orange, 90)      // Orange color if short position.       na(isLong)      => color.new(color.red, 40)         // Red color if no position. Note this line is invalid in v6.   bgcolor(stateColor)      // On the first bar, display the raw value of `isLong` in a table.   if barstate.isfirst       var table t = table.new(position. bottom_right, 2, 4, color.yellow, frame_color = color.black, frame_width = 1)       t.cell(0, 0, "On first bar")       t.cell(0, 1, "`isLong` raw value:", bgcolor = color.new(color.red, 40))       t.cell(1, 1, str.tostring(isLong), bgcolor = color.new(color.red, 40))       // Compare `isLong` value to Boolean `true` and `false` values.       t.cell(0, 2, "`isLong` == `true`?")       t.cell(1, 2, str.tostring(isLong == true))       t.cell(0, 3, "`isLong` == `false`?")       t.cell(1, 3, str.tostring(isLong == false))   ``

**Fix:** Remove any [na()](https://www.tradingview.com/pine-script-reference/v6/#fun_na), [nz()](https://www.tradingview.com/pine-script-reference/v6/#fun_nz), and [fixnan()](https://www.tradingview.com/pine-script-reference/v6/#fun_fixnan) functions that run on “bool” values. Ensure that all “bool” values are correctly interpreted as `true` or `false` states **only**. If your code logic requires a third [na](https://www.tradingview.com/pine-script-reference/v6/#var_na) state to execute as intended, rewrite the code using a different type or structure to achieve the previous three-state behavior.

To adapt our code to Pine v6, we must first remove the following line to resolve the initial compilation error:

`na(isLong)      => color.new(color.red, 40)   `

In v6, the undefined condition (`strategy.position_size == 0`) now returns `false` instead of `na`. Consequently, the script _incorrectly_ highlights the bars where there are _no_ trade positions the same color as those where there are _short_ positions, since `isLong` has the same `false` result for both conditions:

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-No-bool-na-2.BCM2Ys_b_Z1lBgK8.webp)

We want to distinguish between _three_ unique states: long positions, short positions, and no entered positions. Therefore, using a two-state Boolean variable in v6 is no longer suitable. Instead, to maintain our desired behavior, we must _rewrite_ the v6 code to replace the “bool” variable with a different type. For example, we can use an “int” variable to represent our three different `position_size` states using -1, 0, and 1:

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-No-bool-na-3.C2oMRPfl_ZJu6OY.webp)

``//@version=6   strategy("Bool `na` demo v6", overlay=true, margin_long=100, margin_short=100)       // Strategy's long and short trades are based on moving average cross over/under.    longCondition = ta.crossover(ta.sma(close, 14), ta.sma(close, 28))   if (longCondition)       strategy.entry("My Long Entry Id", strategy.long)      shortCondition = ta.crossunder(ta.sma(close, 14), ta.sma(close, 28))   if (shortCondition)       strategy.entry("My Short Entry Id", strategy.short)      //@variable Integer variable that tracks the current direction of the trade.    // Is `-1` when `position_size` is less than 0 (short), `+1` when `position_size` is greater than 0 (long),    //  and `0` when `position_size` is equal to 0 (no trades).   int tradeDirection = if strategy.position_size < 0        -1   else if strategy.position_size > 0        1   else    //strategy.position_size == 0        0      //@variable Background color, set depending on the `tradeDirection`.   color directionColor = switch       tradeDirection == 1  => color.new(color.blue, 90)        // Blue color if long position.       tradeDirection == -1 => color.new(color.orange, 90)      // Orange color if short position.       tradeDirection == 0  =>  na                                // No color if no position.   bgcolor(directionColor)      // On the first bar, display the value of `tradeDirection` in a table for reference.   var table t = table.new(position.bottom_right, 2, 3, color.yellow, frame_color = color.black, frame_width = 1)   if barstate.isfirst       t.cell(0, 0, "On first bar")       t.cell(0, 1, "`tradeDirection` value:", bgcolor = color.new(color.green, 60))       t.cell(1, 1, str.tostring(tradeDirection), bgcolor = color.new(color.green, 60))          //@variable A "string" representation of `tradeDirection` value on current bar.       string directionString = tradeDirection == 1 ? "Long" : tradeDirection == -1 ? "Short" : "No entered positions"       t.cell(0, 2, "State: ")       t.cell(1, 2, directionString)      if barstate.islastconfirmedhistory       //@variable A "string" representation of `tradeDirection`  value on current bar.       string directionString = tradeDirection == 1 ? "Long" : tradeDirection == -1 ? "Short" : "No entered positions"       label.new(bar_index, high, "On last bar \n `tradeDirection` value: " + str.tostring(tradeDirection)             + "\n State: " + directionString)   ``

### [Unique parameters cannot be \`na\`](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#unique-parameters-cannot-be-na)

Some Pine Script™ function parameters expect values of _unique_ types. For example, the `style` parameter of the [plot()](https://www.tradingview.com/pine-script-reference/v6/#fun_plot) function expects a value of the “input plot\_style” qualified type, which must be one of the constants in the `plot.style_*` group.

In v5, passing [na](https://www.tradingview.com/pine-script-reference/v6/#var_na) to the [plot()](https://www.tradingview.com/pine-script-reference/v6/#fun_plot) function’s `style` parameter simply plots a line using the default style `plot.style_line`, without raising an error.

In v6, parameters that expect unique types **no longer** accept [na](https://www.tradingview.com/pine-script-reference/v6/#var_na) values. Additionally, [conditional expressions](https://www.tradingview.com/pine-script-docs/language/conditional-structures/) that return these unique types must be used in a form that **cannot** result in an [na](https://www.tradingview.com/pine-script-reference/v6/#var_na) value. For example, a [switch](https://www.tradingview.com/pine-script-reference/v6/#kw_switch)\-statement must have a `default` block, and an [if](https://www.tradingview.com/pine-script-reference/v6/#kw_if)\-statement must have an `else`\-block, because these conditional expressions can return [na](https://www.tradingview.com/pine-script-reference/v6/#var_na) otherwise.

The following example script shows two code structures that work in v5 but raise errors in v6.

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-No-enum-na.Rirf8rCY_ZFlyGA.webp)

``//@version=5   indicator("`na` and unique types demo v5")      //@variable User-selected "string" to determine type of plot used for `plot()` function's `style` argument.   string inputStyle = input.string("Area", "Plot style", options = ["Area", "Columns", "Histogram", "Stepline-diamond"])      // Initialize an `input plot_style` type variable based on user's selected `inputStyle`.   selectedPlotStyle = switch inputStyle       "Area"             => plot.style_area       "Columns"          => plot.style_columns       "Histogram"        => plot.style_histogram       "Stepline-diamond" => plot.style_stepline_diamond   // `switch` statement covers all `inputStyle` options, but does not include `default` block.   // Valid in v5. Invalid in v6 - `switch` statement must include a `default` block, otherwise raises error.      plot(close, "Source plot", color.blue, 2, style = selectedPlotStyle)      //@variable Toggle for the style of the line plotted at price "100".   inputHundredStyle = input.bool(true, " Use crosses style for '100-line'")      hundredLineStyle = if inputHundredStyle       plot.style_cross   // Since there is no `else` block, setting `inputHundredStyle` to `false` makes this variable `na`.   // In v5, passing `na` to the `style` parameter makes the `plot()` function use its default style `plot.style_line`.   // In v6, this raises a compilation error because `style` cannot be `na`.      // Plot the "100-line" using the `hundredLineStyle` style constant.   plot(100, "100-line", color.orange, 4, style = hundredLineStyle)   ``

**Fix:** Ensure that no [na](https://www.tradingview.com/pine-script-reference/v6/#var_na) value is passed to parameters that expect unique types, and that all conditional statements return a suitable non-[na](https://www.tradingview.com/pine-script-reference/v6/#var_na) value.

``//@version=6   indicator("`na` and unique types demo v6")      //@variable User-selected "string" to determine type of plot used for `plot()` function's `style` argument.   string inputStyle = input.string("Area", "Plot style", options = ["Area", "Columns", "Histogram", "Stepline-diamond"])      // Initialize an `input plot_style` type variable based on user's selected `inputStyle`.   selectedPlotStyle = switch inputStyle       "Area"             => plot.style_area       "Columns"          => plot.style_columns       "Histogram"        => plot.style_histogram       "Stepline-diamond" => plot.style_stepline_diamond       // A default block must be included in v6.       =>  plot.style_line      plot(close, "Source plot", color.blue, 2, style = selectedPlotStyle)      //@variable Toggle for the style of the line plotted at price "100".   inputHundredStyle = input.bool(true, " Use crosses style for '100-line'")      hundredLineStyle = if inputHundredStyle       plot.style_cross   else    //`else` block must be included in v6. Sets "line" style if `inputHundredStyle` is `false`.       plot.style_line      // Plot the "100-line" using the `hundredLineStyle` style constant.   plot(100, "100-line", color.orange, 4, style = hundredLineStyle)   ``

[Constants](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#constants)
-------------------------------------------------------------------------------------------------------

The following changes have been made to how Pine handles constant values.

### [Fractional division of constants](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#fractional-division-of-constants)

Dividing two integer “const” values can return a fractional value.

In v5, the result of the division of two “int” values is inconsistent. If _both_ values are qualified as “const”, the script performs what is known as _integer division_, and discards any fractional remainder in the result, e.g., `5/2 = 2`. However, if _at least one_ of the integers is qualified as “input”, “simple”, or “series”, the script _preserves_ the fractional remainder in the division result: `5/2 = 2.5`.

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-No-int-division-1.vzmEC1NZ_ZLqSka.webp)

``//@version=5   indicator("`int` division demo")      // `float` division produces fractional remainder in both v5 and v6.   plot( 5.0 / 2.0, "`float` values", color.blue)      // `const int` division produces rounded-down result in v5. In v6, it produces a fractional remainder.   plot( 5 / 2,           "`const int` values",     color.orange)   plot( int(5) / int(2), "values wrapped `int()`", color.red)      // Wrapped `int()` division produces rounded down result in both v5 and v6.   plot( int(5 / 2), "result wrapped `int()`", color.green)      // Using `input int` type in division preserves the fractional remainder in both v5 and v6.   inputNum = input.int(2, "Division int", minval = 1)   plot( 5 / inputNum, "`input int` value", color.purple)   ``

In v6, dividing two “int” values that are not evenly divisible _always_ results in a number with a _fractional value_, regardless of the type and qualifier of the two arguments used. Therefore, the v6 division result is `5/2 = 2.5`, even if both values involved are “const int”.

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-No-int-division-2.DNd_hajv_1TwdjI.webp)

**Fix:** If you need an “int” division result _without_ a fractional value, wrap the division with the [int()](https://www.tradingview.com/pine-script-reference/v6/#fun_int) function to cast the _result_ to “int”, which discards the fractional remainder. Alternatively, use [math.round()](https://www.tradingview.com/pine-script-reference/v6/#fun_math.round), [math.floor()](https://www.tradingview.com/pine-script-reference/v6/#fun_math.floor), or [math.ceil()](https://www.tradingview.com/pine-script-reference/v6/#fun_math.ceil) to _round_ the division result in a specific direction.

### [Mutable variables are always “series”](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#mutable-variables-are-always-series)

In Pine v5, some mutable variables are qualified as “series” values but are _erroneously_ qualified as “const”. This behavior is incorrect and allows a programmer to pass them where “series” variables are usually not accepted.

For example, the [ta.ema()](https://www.tradingview.com/pine-script-reference/v6/#fun_ta.ema) function expects its `length` argument to be an integer qualified as “simple” or weaker (see the [Qualifiers hierarchy](https://www.tradingview.com/pine-script-docs/language/type-system/#qualifiers)). In the example script below the `seriesLen` variable is effectively a “series” type because its value changes between bars. In v5, `seriesLen` _can_ be passed to [ta.ema()](https://www.tradingview.com/pine-script-reference/v6/#fun_ta.ema). Although this does not raise an error, it does not work as expected, because only its _first_ recorded value `1` is used as the `length` in the script:

``//@version=5   indicator("`const` mutable variables demo")      // Variable is effectively of `series int` type.    var seriesLen = 0   seriesLen += 1      // `ta.ema()` only uses `length = 1` throughout execution, even as `seriesLen` changes.   plot(ta.ema(close, seriesLen))   ``

In v6, `seriesLen` is correctly parsed as a “series int” type, and raises a compilation error if passed in place of the expected “simple int” argument for `length`.

**Fix:** Pass values of the expected qualified type to built-in functions. In our example, set the `length` argument to a “const int” value.

``//@version=6   indicator("`const` mutable variables demo")      // Variable is now of `const int` type.   var seriesLen = 1      // `ta.ema()` uses `length = 1` throughout execution.   plot(ta.ema(close, seriesLen))   ``

### [Color changes](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#color-changes)

The color values behind some of the `color.*` constants have changed in Pine v6 to better reflect the TradingView palette:

| Constant name | Pine v5 color | Pine v6 color |
| --- | --- | --- |
| color.red | #FF5252 | #F23645 |
| color.teal | #00897B | #089981 |
| color.yellow | #FFEB3B | #FDD835 |

Additionally, the default text color for [label.new()](https://www.tradingview.com/pine-script-reference/v6/#fun_label.new) is now `color.white` in v6 (previously `color.black` in v5) to ensure that the text is more visible against the default `color.blue` label.

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-Color-changes.BNSaJyQ-_Zq4WdJ.webp)

`//@version=6   indicator("Default colors v6")      color defaultColor = switch       bar_index == last_bar_index     => color.yellow       bar_index == last_bar_index - 1 => color.green       bar_index == last_bar_index - 2 => color.red       => na   bgcolor(defaultColor)      if barstate.islastconfirmedhistory       label.new(bar_index + 2, 0, "Default text color")   `

[Strategies](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#strategies)
---------------------------------------------------------------------------------------------------------

### [Removal of \`when\` parameter](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#removal-of-when-parameter)

The `when` parameter for order creation functions was deprecated in v5 and is removed in v6. An order is created only if the `when` condition is `true`, which is its default value. This parameter affects the following functions: [strategy.entry()](https://www.tradingview.com/pine-script-reference/v6/#fun_strategy.entry), [strategy.order()](https://www.tradingview.com/pine-script-reference/v6/#fun_strategy.order), [strategy.exit()](https://www.tradingview.com/pine-script-reference/v6/#fun_strategy.exit), [strategy.close()](https://www.tradingview.com/pine-script-reference/v6/#fun_strategy.close), [strategy.close\_all()](https://www.tradingview.com/pine-script-reference/v6/#fun_strategy.close_all), [strategy.cancel()](https://www.tradingview.com/pine-script-reference/v6/#fun_strategy.cancel), and [strategy.cancel\_all()](https://www.tradingview.com/pine-script-reference/v6/#fun_strategy.cancel_all).

The following example strategy shows the use of the `when` parameter, and works in v5 but not v6.

`//@version=5   strategy("Conditional strategy", overlay=true)      longCondition = ta.crossover(ta.sma(close, 14), ta.sma(close, 28))   strategy.entry("My Long Entry Id", strategy.long, when = longCondition)      shortCondition = ta.crossunder(ta.sma(close, 14), ta.sma(close, 28))   strategy.entry("My Short Entry Id", strategy.short, when = shortCondition)   `

**Fix:** To trigger the order creation conditionally, use [if](https://www.tradingview.com/pine-script-reference/v6/#kw_if) statements instead.

`//@version=6   strategy("Conditional strategy", overlay=true)      longCondition = ta.crossover(ta.sma(close, 14), ta.sma(close, 28))   if longCondition       strategy.entry("My Long Entry Id", strategy.long)      shortCondition = ta.crossunder(ta.sma(close, 14), ta.sma(close, 28))   if shortCondition       strategy.entry("My Short Entry Id", strategy.short)   `

### [Default margin percentage](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#default-margin-percentage)

The default margin percentage for strategies is now 100.

In v5, the default value of the `margin_long` and `margin_short` parameters is 0, which means that the strategy **does not check** its available funds before creating or managing orders. It can create orders that require _more_ money than is available, and will **not** close short orders even when they lose more money than available to the strategy.

In _Pine v6_, the default margin percentage is 100. The strategy **does not open** entries that require more money than is available, and short orders are _margin called_ if too much money is lost.

For example, we can see the difference in strategy behavior by running this simple strategy on the “ARM” symbol’s 4h chart using the v5 and v6 default margin values. When using Pine v5, there are no margin calls:

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-Margin-calls-1.CnzpPLkm_1lRc8h.webp)

`//@version=5   strategy("My strategy", overlay=true, default_qty_type = strategy.percent_of_equity, default_qty_value=100)   // v6 defaults: margin_long=100, margin_short=100    // v5 defaults: margin_long=0, margin_short=0      longCondition = ta.crossover(ta.sma(close, 14), ta.sma(close, 28))   if (longCondition)       strategy.entry("My Long Entry Id", strategy.long)      shortCondition = ta.crossunder(ta.sma(close, 14), ta.sma(close, 28))   if (shortCondition)       strategy.entry("My Short Entry Id", strategy.short)   `

However, if we adjust this script to `//@version=6` on the same chart, we see that it triggers 14 margin calls because of the new margin percentages:

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-Margin-calls-2.DY4IZ4xs_ZukbpG.webp)

**Fix:** To replicate the previous v5 behavior, set the [strategy()](https://www.tradingview.com/pine-script-reference/v6/#fun_strategy) function’s `margin_short` and `margin_long` arguments to 0.

### [Excess orders are trimmed](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#excess-orders-are-trimmed)

Strategy orders above the 9000 limit are trimmed (removed) in v6.

In v5, outside of Deep Backtesting, when a strategy creates more than 9000 orders, it raises a runtime error and halts any further calculations.

For example, this strategy script places several orders on each bar in the dataset. As a result, it can quickly surpass the 9000 order limit and trigger an error in Pine v5:

`//@version=5   strategy("Strategy order limit demo", overlay=true, pyramiding=5)      // Place several long orders on every even bar. This reaches the maximum orders limit in v5 and raises a runtime error.   if bar_index % 2 == 0       for i = 1 to 5           strategy.entry("Entry " + str.tostring(i), strategy.long, qty = 5)   // Place short orders on every odd bar.   else       strategy.entry("Short", strategy.short, qty = 25)   `

In v6, when the total number of orders exceeds 9000, the strategy does _not_ halt. Instead, the orders are _trimmed_ from the beginning until the limit is reached, meaning that the strategy only stores the information for the most recent orders.

Trimmed orders no longer show in the Strategy Tester, and referencing them using the `strategy.closedtrades.*` functions returns [na](https://www.tradingview.com/pine-script-reference/v6/#var_na). Use [strategy.closedtrades.first\_index](https://www.tradingview.com/pine-script-reference/v6/#var_strategy.closedtrades.first_index) to get the index of the first _non-trimmed_ trade:

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-Order-trimming.CFIVLCGx_Z2iQqmO.webp)

``//@version=6   strategy("Strategy order limit demo", overlay=true, pyramiding=5)      //@variable Count of total orders placed.   var int totalOrders = 0      // Place several long orders on every even bar.   if bar_index % 2 == 0       for i = 1 to 5           strategy.entry("Entry " + str.tostring(i), strategy.long, qty = 5)           totalOrders += 1   // Place short orders on every odd bar.   else       strategy.entry("Short", strategy.short, qty = 25)       totalOrders += 1      // Display total orders and index of first non-trimmed trade in a table cell on last bar.   if barstate.islastconfirmedhistory       var table t = table.new(position.bottom_right, 1, 3, color.yellow, color.black, 1)       // Display total orders and closed trades counts.       string ordersText = "Total orders: " + str.tostring(totalOrders, "#,###")            + "\n Closed trades: " + str.tostring(strategy.closedtrades, "#,###")       t.cell(0, 0, ordersText, text_halign = text.align_right, text_size = size.large)          // Display the first non-trimmed trade index and its entry price.       string firstTradeIndex = str.tostring(strategy.closedtrades.first_index, "#,###")       string firstTradePrice = str.tostring(strategy.closedtrades.entry_price(strategy.closedtrades.first_index), "$##.##")       string firstTradeText = str.format("Index of first non-trimmed trade: {0}\nEntry price of trade #{0}: {1}", firstTradeIndex, firstTradePrice)       t.cell(0, 1, firstTradeText, text_halign = text.align_right, text_size = size.large, bgcolor = #61dd5165)          // Trying to reference the trimmed trades (e.g., first closed trade) returns `na`.       if totalOrders > 9000           string trimmedTradePrice = "Entry price of trade #0: " + str.tostring(strategy.closedtrades.entry_price(0))           t.cell(0, 2, trimmedTradePrice, text_size = size.large, bgcolor = #dd51c665)   ``

### [\`strategy.exit()\` evaluates parameter pairs](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#strategyexit-evaluates-parameter-pairs)

The [strategy.exit()](https://www.tradingview.com/pine-script-reference/v6/#fun_strategy.exit) function has three sets of _relative_ and _absolute_ parameters that define price levels for exit order calculations. The relative parameters `profit`, `loss`, and `trail_points` specify the [take-profit](https://www.tradingview.com/pine-script-docs/concepts/strategies/#take-profit-and-stop-loss) and [stop-loss](https://www.tradingview.com/pine-script-docs/concepts/strategies/#take-profit-and-stop-loss) levels and [trailing stop](https://www.tradingview.com/pine-script-docs/concepts/strategies/#trailing-stops) activation level as _tick distances_ from the entry price. In contrast, the absolute parameters `limit`, `stop`, and `trail_price` specify the exit and trail activation _prices_ directly.

In Pine v5, a [strategy.exit()](https://www.tradingview.com/pine-script-reference/v6/#fun_strategy.exit) call containing arguments for both the relative and absolute parameters that define a price level for the same exit order always prioritizes the _absolute_ parameter and ignores the relative one. For instance, a call that includes a `limit` and `profit` argument consistently places take-profit orders at the `limit` value. It never places an exit order using the `profit` distance.

In Pine v6, if a [strategy.exit()](https://www.tradingview.com/pine-script-reference/v6/#fun_strategy.exit) call contains arguments for related absolute and relative parameters, it evaluates _both_ specified levels and uses the one that the market price is expected to _trigger first_.

The example below demonstrates how the behavior of this command differs for v5 and v6 scripts. This v5 script creates a long [market order](https://www.tradingview.com/pine-script-docs/concepts/strategies/#market-orders) and an exit order [bracket](https://www.tradingview.com/pine-script-docs/concepts/strategies/#take-profit-and-stop-loss) on each 28th bar. The [strategy.exit()](https://www.tradingview.com/pine-script-reference/v6/#fun_strategy.exit) call contains arguments for the relative parameters that determine take-profit and stop-loss levels (`profit` and `loss`), and it includes arguments for the absolute parameters (`limit` and `stop`). The `profit` and `loss` arguments are both 0, which would result in consistent exits at the entry price if the command used them. However, the command never uses these values to determine the exit order levels because the `limit` and `stop` parameters _always_ take precedence when they have specified values:

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/To-pine-version-6-Strategies-Strategy-exit-evaluates-parameter-pairs-1.Bq0Yb_zb_jLvAa.webp)

``//@version=5   strategy("`strategy.exit()` with parameter pairs demo", overlay = true, margin_long = 100, margin_short = 100)      //@variable The 14-bar Average True Range.    float atr = ta.atr(14)      if bar_index % 28 == 0       strategy.entry("Buy", strategy.long)       strategy.exit("Exit", "Buy", profit = 0, limit = close + 2.0 * atr, loss = 0, stop = close - 2.0 * atr)   ``

If we convert the script to Pine v6, its behavior changes. Instead of prioritizing the absolute `limit` and `stop` parameters exclusively, the [strategy.exit()](https://www.tradingview.com/pine-script-reference/v6/#fun_strategy.exit) command always prioritizes the price levels that will trigger exits _first_. In this example, the market price reaches the `limit` or `stop` value _after_ the `profit` and `loss` distance of 0 ticks. Consequently, the command ignores the `limit` and `stop` values and places its exit orders at the entry price, which causes the strategy to exit each trade immediately after opening it:

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/To-pine-version-6-Strategies-Strategy-exit-evaluates-parameter-pairs-2.Q464KiWa_Z2rlE7N.webp)

``//@version=6   strategy("`strategy.exit()` with parameter pairs demo", overlay = true, margin_long = 100, margin_short = 100)      //@variable The 14-bar Average True Range.    float atr = ta.atr(14)      if bar_index % 28 == 0       strategy.entry("Buy", strategy.long)       strategy.exit("Exit", "Buy", profit = 0, limit = close + 2.0 * atr, loss = 0, stop = close - 2.0 * atr)   ``

[History-referencing operator](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#history-referencing-operator)
---------------------------------------------------------------------------------------------------------------------------------------------

Pine v6 contains several changes to referencing the history of values.

### [No history for literal values](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#no-history-for-literal-values)

The history-referencing operator `[]` can no longer be used with literal values or built-in constants.

In v5, the history-referencing operator `[]` can be used with built-in constants, such as `true` and `color.red`, and with _literal_s, which are raw values used directly in a script that are not stored as variables, such as `6` or `"myString"`, etc.

However, referencing the history of a literal is usually redundant, because by definition every literal represents a fixed value. The only exception where the returned historic value may vary is if the historical offset points to a _non-existent_ bar, in which case referencing the historic literal value returns [na](https://www.tradingview.com/pine-script-reference/v6/#var_na).

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-History-for-literals-1.CQYj6Sq2_ZChn43.webp)

``//@version=5   indicator("History-referencing on literals demo")      // These lines all use history-referencing on literals, which works in v5, but is not really useful to do here.    plot(6[1], "6[1]", linewidth = 3)   bgcolor(true[10] ? color.orange[3] : na)   if barstate.islastconfirmedhistory       // Since "string literal" is only defined in the last bar scope, history-referencing here returns `na`.       labelText = "string literal"[20]       label.new(bar_index - 3, 3, labelText + ", more text", textcolor = color.white, size = size.large)       // Label output will only show ", more text" in v5, since `labelText` is `na`.      // In v6, using any history-referencing on literals or built-in constants causes an error.   ``

In _Pine v6_, you can **no longer** use the history-referencing operator `[]` on literals or built-in constants. Trying to do so triggers a compilation error.

**Fix:** Remove any `[]` operators used with literals or constants.

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-History-for-literals-2.DnTquvm7_1nXvDJ.webp)

``//@version=6   indicator("History-referencing on literals demo")      // We no longer use history-referencing on literals in v6.    plot(6, "6", linewidth = 3)   bgcolor(true ? color.orange : na)   if barstate.islastconfirmedhistory       labelText = "string literal"       label.new(bar_index - 3, 3, labelText + ", more text", textcolor = color.white, size = size.large)       // Label output shows "string literal, more text" in v6, since `labelText` is defined without history-referencing anymore.   ``

### [History of UDT fields](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#history-of-udt-fields)

The history-referencing operator `[]` can no longer be used directly on fields of user-defined types.

In v5, you can use the history-referencing operator `[]` on the _fields_ of [_user-defined types_](https://www.tradingview.com/pine-script-docs/language/objects/). While this does not cause any compilation errors, the behavior itself is erroneous.

For example, the script below draws an arrow [label](https://www.tradingview.com/pine-script-reference/v6/#type_label) on each bar and displays its percentage increase/decrease. The label style, color, and text are set based on a bar’s direction (`close > open`). The script defines a [UDT](https://www.tradingview.com/pine-script-docs/language/objects/) `LblSettings` to initialize an object on each bar that stores these settings. On the last bar, it draws a [table](https://www.tradingview.com/pine-script-reference/v6/#type_table) cell that displays the arrow direction and percentage difference from 10 bars back. In v5, we could use the history-referencing operator `[]` on the required `LblSettings` fields directly:

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-History-for-UDTs.h94G4ccp_wdX75.webp)

``//@version=5   indicator("UDT history-referencing demo", overlay = true)      //@type A custom type to hold bar's `label` settings based on bar's direction.   //Includes bar direction, label style and color, and "string" percentage difference between bar's `open` and `close`.    type LblSettings       bool isUp = false       string lblStyle       color lblColor       string diff      //@variable A `LblSettings` instance declared on every bar.    LblSettings infoObject = LblSettings.new()      // Set the `LblSettings` object fields based on current bar's direction and price information.   infoObject.isUp := close > open   infoObject.lblStyle := infoObject.isUp ? label.style_arrowup : label.style_arrowdown   infoObject.lblColor := infoObject.isUp ? color.green : color.red   infoObject.diff := str.tostring((close - open) / open * 100, "#.##") + "%"      // Display a new `label` on each bar using its `infoObject` settings.   label.new(bar_index, high, infoObject.diff, style = infoObject.lblStyle,         color = infoObject.lblColor, textcolor = infoObject.lblColor)      // Highlight the bar that is 10 bars back from the last bar.   bgcolor(bar_index == last_bar_index - 10 ? color.yellow : na)      // On last bar, output table cell to display `LblSettings` object's `lblStyle` and `diff` fields from 10 bars back.    if barstate.islast       var table t = table.new(position.bottom_right, 1, 1, color.yellow)          // In v5, you could use history-referencing operator `[]` on UDT fields directly.        //@variable Text displayed in table cell. Set based on the `lblStyle` and `diff` fields from 10 bars back.       string txt = "10 bars back: Arrow was "             + (infoObject.lblStyle[10] == label.style_arrowdown ? "DOWN" : "UP")            + " by " + infoObject.diff[10]       t.cell(0, 0, txt, text_size = size.large)   ``

In _Pine v6_, you can **no longer** use the history-referencing operator `[]` on the field of a user-defined type directly.

**Fix:** Use the history-referencing operator on the UDT _object_ instead, then retrieve the field of the historic object. To do so, use the syntax `(myObject[10]).field` - ensure the object’s historical reference is wrapped in _parentheses_, otherwise it is invalid. Alternatively, assign the UDT _field_ to a _variable_ first, and then use the history-referencing operator `[]` on the variable to access its historic value.

`// Reference history of object, then retrieve field of historic object.   [fieldType] historicFieldValue = (myObject[10]).field      // Alternative: Assign field to variable, then reference history of variable to get historic field value.   [fieldType] newVariable = myObject.field   [fieldType] historicFieldValue = newVariable[10]   `

Therefore, we can adjust the v5 code to access a historic instance of our `infoObject` on the last bar, wrapped in parentheses. Then, we retrieve our desired field values from the historic object `(infoObject[10])` to display the arrow direction and percentage difference from 10 bars back:

``//@version=6   indicator("UDT history-referencing demo", overlay = true)      //@type A custom type to hold bar's `label` settings based on bar's direction.   //Includes bar direction, label style and color, and "string" percentage difference between bar's `open` and `close`.    type LblSettings       bool isUp = false       string lblStyle       color lblColor       string diff      //@variable A `LblSettings` instance declared on every bar.    LblSettings infoObject = LblSettings.new()      // Set the `LblSettings` object fields based on current bar's direction and price information.   infoObject.isUp := close > open   infoObject.lblStyle := infoObject.isUp ? label.style_arrowup : label.style_arrowdown   infoObject.lblColor := infoObject.isUp ? color.green : color.red   infoObject.diff := str.tostring((close - open) / open * 100, "#.##") + "%"      // Display a new `label` on each bar using its `infoObject` settings.   label.new(bar_index, high, infoObject.diff, style = infoObject.lblStyle,         color = infoObject.lblColor, textcolor = infoObject.lblColor)      // Highlight the bar that is 10 bars back from the last bar.   bgcolor(bar_index == last_bar_index - 10 ? color.yellow : na)      // On last bar, output table cell to display `LblSettings` object's `lblStyle` and `diff` fields from 10 bars back.    if barstate.islast       var table t = table.new(position.bottom_right, 1, 1, color.yellow)          // In v6, cannot use `[]` on UDT fields (e.g., `infoObject.lblStyle[10]` is invalid).       // Instead, Use `[]` to reference UDT object's history, wrapped in parentheses, then retrieve its fields.       //@variable The `lblStyle` field value from 10 bars back. Is either `label.style_arrowdown` or `label.style_arrowup`.       string historicArrowStyle = (infoObject[10]).lblStyle       //@variable The `diff` field value (percentage difference between `close` and `open`) from 10 bars back.       string historicPercentDifference = (infoObject[10]).diff          //@variable Text displayed in table cell. Set based on the `lblStyle` and `diff` fields from 10 bars back.       string txt = "10 bars back: Arrow was "             + (historicArrowStyle == label.style_arrowdown ? "DOWN" : "UP")            + " by " + historicPercentDifference       t.cell(0, 0, txt, text_size = size.large)   ``

[Timeframes must include a multiplier](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#timeframes-must-include-a-multiplier)
-------------------------------------------------------------------------------------------------------------------------------------------------------------

The [timeframe.period](https://www.tradingview.com/pine-script-reference/v6/#var_timeframe.period) variable holds a “string” that represents the chart’s timeframe, typically consisting of a _quantity_ (multiplier) and _unit_.

In v5, the [timeframe.period](https://www.tradingview.com/pine-script-reference/v6/#var_timeframe.period) variable does _not_ include a quantity when the chart timeframe has a multiplier of `1`. Instead, the string consists of only the timeframe unit, e.g., `"D"`, `"W"`, `"M"`. This is inconsistent with the timeframe strings for these same units at higher intervals, e.g., `"2D"`, `"3M"`.

To simplify the timeframe format in v6, the [timeframe.period](https://www.tradingview.com/pine-script-reference/v6/#var_timeframe.period) variable now _always_ includes a multiplier with its timeframe unit. So, `"D"` becomes `"1D"`, `"W"` becomes `"1W"`, and `"M"` becomes `"1M"`.

This change might affect the behavior of older scripts that used `==` to compare the value of [timeframe.period](https://www.tradingview.com/pine-script-reference/v6/#var_timeframe.period) with the “string” representation of a timeframe directly (e.g., `timeframe.period == "D"`).

To show the difference between the v5 and v6 [timeframe.period](https://www.tradingview.com/pine-script-reference/v6/#var_timeframe.period) variables, we ran the script below on a daily chart (1D) for each Pine version. The script displays the [timeframe.period](https://www.tradingview.com/pine-script-reference/v6/#var_timeframe.period) string in a [table](https://www.tradingview.com/pine-script-reference/v6/#type_table), and compares the variable’s value with the “string” literals `"D"` and `"1D"`:

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-Timeframe-multiplier.C0vAAkQ8_ZbC7c2.webp)

``//@version=6   indicator("`timeframe.period` multiplier - v6")      //@function Compares `timeframe.period` to passed `timeframeString` and outputs result in selected table cell.   compareTF(string timeframeString, table t, int row) =>       bool tfComparison = timeframe.period == timeframeString       // Format table cell text and determine cell color based on comparison result.       string displayText = "`timeframe.period` == '" + timeframeString + "'? " + str.tostring(tfComparison)       color cellColor = tfComparison ? color.rgb(76, 175, 79, 40) : color.rgb(255, 82, 82, 40)       // Display `tfComparison` result.       t.cell(0, row, displayText, bgcolor = cellColor, text_halign = text.align_left, text_size = size.large)      // Display the chart's timeframe information in a table.   if barstate.islastconfirmedhistory       //@variable Table displaying chart timeframe information.       var table t = table.new(position.middle_center, 1, 3, #FFEB3B99, border_color = color.black, border_width = 1)          //@variable The text to display in the table, consisting of the chart timeframe and multiplier.       string tfInfo = "Chart timeframe: " + timeframe.period            + "\n Chart multiplier: " + str.tostring(timeframe.multiplier)           t.cell(0, 0, tfInfo, text_halign = text.align_left, text_size = size.large)          // Compare the current chart timeframe (daily chart) to timeframe strings with and without a multiplier.       compareTF("D",  t, 1)       compareTF("1D", t, 2)   ``

**Fix:** In general, ensure that all timeframe strings include a multiplier. In this example, change the timeframe comparison “string” (`timeframe.period == "D"`) to ensure the “string” literal includes a multiplier (`timeframe.period == "1D"`).

[Lazy evaluation of conditions](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#lazy-evaluation-of-conditions)
-----------------------------------------------------------------------------------------------------------------------------------------------

The `and` and `or` conditions are now evaluated _lazily_ rather than _strictly_.

An `and` condition is `true` if _all_ of its arguments are `true`, which means that if the _first_ argument is `false`, we can deduce that the whole condition is `false`, regardless of the value of the second argument. Conversely, an `or` condition is `true` when _at least one_ of the arguments is `true`, so if the _first_ argument is already `true`, then the whole condition is `true`, regardless of the second argument’s state.

Pine v5 evaluates all [bool](https://www.tradingview.com/pine-script-reference/v6/#type_bool) expressions except for the `?:` [ternary operator](https://www.tradingview.com/pine-script-docs/language/operators/#-ternary-operator) _strictly_, meaning the _second_ part of a conditional expression is _always_ evaluated, regardless of the value of the first argument.

Lazy evaluation can have consequences for script calculation. In the example below, we assign a value of `true` to the `signal` variable _only_ when `close > open` _and_ `ta.rsi(close, 14) > 50`. The [ta.rsi()](https://www.tradingview.com/pine-script-reference/v6/#fun_ta.rsi) function [must be executed on every bar](https://www.tradingview.com/pine-script-docs/language/execution-model/#historical-values-of-functions) in order to calculate its result correctly. In v5, the function _is_ called on every bar, even when `close > open` is _not_ `true`, due to the strict bool evaluation, and therefore the function calculates correctly.

``//@version=5   indicator("Evaluation test v5")      //@variable A signal flag. Is `true` if two conditions `close > open` and `ta.rsi(close, 14) > 50` are both `true`.   bool signal = false      if close > open and ta.rsi(close, 14) > 50       signal := true      // Highlight background on bars where `signal` is `true`.   bgcolor(signal ? color.new(color.green, 90) : na)   ``

In v6, [bool](https://www.tradingview.com/pine-script-reference/v6/#type_bool) expressions are evaluated _lazily_, which means the expression _stops evaluating_ once it determines the overall condition’s result, even if there are other arguments remaining in the expression.

If we convert the script above to v6, we see that the plotted signals _differ_ between the two scripts. This variation occurs because of the lazy bool evaluation – since an `and` condition is only `true` if _all_ its arguments are `true`, when `close > open` is `false`, the `and` condition is _definitely_ `false` regardless of the second argument `ta.rsi(close, 14) > 50`. Consequently, the [ta.rsi()](https://www.tradingview.com/pine-script-reference/v6/#fun_ta.rsi) call is _not_ evaluated on every bar, which interferes with the internal history that the RSI function stores for its calculation and results in incorrect values:

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-Lazy-evaluation.DTwy6SAe_64SBs.webp)

**Fix:** Ensure that the script evaluates all functions that rely on previous values on each bar. For example, extract calls that rely on historical context to the _global scope_ and assign them to a variable. Then, reference that _variable_ in the `and` and `or` conditions.

Note that you can and should take advantage of the lazy bool evaluation to create smarter, more concise code.

For example, the script below calls [array.first()](https://www.tradingview.com/pine-script-reference/v6/#fun_array.first) on an [array](https://www.tradingview.com/pine-script-reference/v6/#type_array) that is occasionally empty (on bars where `close > open` is `false`). In _Pine v5_, calling [array.first()](https://www.tradingview.com/pine-script-reference/v6/#fun_array.first) on an empty array results in a runtime error, so you must keep the two [if](https://www.tradingview.com/pine-script-reference/v6/#kw_if)\-conditions that check the array size and first element separated in _different scopes_ to avoid the error. However, in _Pine v6_, you can have the two conditions in the _same_ _scope_ without error because the `and` condition’s lazy evaluation ensures that [`array.first()`](https://www.tradingview.com/pine-script-reference/v6/#fun_array.first) will only be called if `array.size() != 0` is `true` first:

``//@version=5   indicator("Lazy evaluation error showcase")      array<bool> myArray = array.new<bool>()      if close > open       myArray.push(true)      // Causes a runtime error in v5 when trying to call `array.first()` on an empty array.   // Works in v6 because `array.first()` is only called if the array is not empty.   if myArray.size() != 0 and myArray.first()       label.new(bar_index, high, "Test")      // A correct approach for v5: `array.first()` is only called when we're sure the array is not empty.   if myArray.size() != 0        if myArray.first()           label.new(bar_index, high, "Test")   ``

[Cannot repeat parameters](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#cannot-repeat-parameters)
-------------------------------------------------------------------------------------------------------------------------------------

In v5, you can specify the same parameter in a function more than once. However, doing so raises a _compiler warning_, and only the _first_ value will be used.

``// In v5, compiles but raises warning. Only uses first value, so plot color will be `blue`.   plot(close, "Close", color = color.blue, linewidth = 2, color = color.red)   ``

In v6, you can specify a parameter only _once_, and doing otherwise will result in a _compilation error_.

**Fix:** Remove the duplicate parameters.

`// In v6, script will not compile if parameter is specified more than once.   plot(close, "Close", color = color.blue, linewidth = 2)   `

[No series \`offset\` values](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#no-series-offset-values)
---------------------------------------------------------------------------------------------------------------------------------------

The `offset` parameter can no longer accept “series” values

In Pine v5, the `offset` parameter in [plot()](https://www.tradingview.com/pine-script-reference/v6/#fun_plot) and similar functions can accept “_series_ int” arguments. However, passing a “series” argument raises a compiler warning, and the behavior is _incorrect_: only the _last_ calculated offset is used on the whole chart, regardless of its previous values.

For example, this script uses `bar_index / 2` as a “series” `offset` argument while plotting the high points of each bar’s body. Because the [plot()](https://www.tradingview.com/pine-script-reference/v6/#fun_plot) function uses only the _last_ `offset` value, the plot appears offset by 10 bars here for the _entire_ “GOOGL” 12M chart (since the chart’s last `bar_index` is 20 here):

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-No-series-offset.DH-Z-RyE_Z1mLK8f.webp)

``//@version=5   indicator("`offset` parameter demo", overlay = true)      //@variable `series int` value. Used as `offset` parameter value in `plot()`.   int seriesOffset = bar_index / 2   // In v5, a `series` type `offset` value is valid, but only the last calculated value is used.     plot(math.max(close, open),"", color.orange, 4, plot.style_stepline, offset = seriesOffset)   ``

In v6, the `offset` parameter accepts an argument qualified as “_simple_” or weaker. The value used must be the same on every bar.

Remember that the Pine Script™ [qualifiers](https://www.tradingview.com/pine-script-docs/language/type-system/#qualifiers) hierarchy means that a parameter expecting a “[simple](https://www.tradingview.com/pine-script-docs/language/type-system/#simple)” value can also accept values qualified as “[input](https://www.tradingview.com/pine-script-reference/v6/#type_input)” or “[const](https://www.tradingview.com/pine-script-reference/v6/#type_const)”. However, passing a “series” argument triggers a compilation error.

**Fix**: Change any “series” values passed to `offset` to “simple” values.

[Minimum \`linewidth\` is 1](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#minimum-linewidth-is-1)
-------------------------------------------------------------------------------------------------------------------------------------

In v5, the `linewidth` parameter of the [plot()](https://www.tradingview.com/pine-script-reference/v6/#fun_plot) and [hline()](https://www.tradingview.com/pine-script-reference/v6/#fun_hline) functions can accept a value smaller than 1, although the width on the chart will still appear as 1 for these drawings:

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-Minimum-linewidth-1.DK4UaZ1K_ZBnFll.webp)

``//@version=5   indicator("Linewidth demo")      //@variable User-input width for a line. Default value set to 0, with no minimum limit.   int userWidth = input.int(0, "Linewidth")      // Valid in v5, but line widths on chart all appear as `linewidth=1`. Not valid in v6.   plot(close,      "LW 1",   linewidth = 1)   plot(close + 5,  "LW 0",   linewidth = userWidth)   plot(close + 10, "LW -5",  linewidth = -5)   hline(240, "hline", color.maroon, linewidth = -3)   ``

In v6, the `linewidth` argument **must** be 1 or greater. Passing a smaller value causes a compilation error.

**Fix:** Replace any `linewidth` argument that is smaller than 1 to ensure all width values are _at least_ 1.

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-Minimum-linewidth-2.BTZsUVkx_VCJPf.webp)

`//@version=6   indicator("Linewidth demo")      //@variable User-input width for a line. Default value set to 2, with minimum value set to 1.   int userWidth = input.int(2, "Linewidth", minval = 1)      // In v6, all line widths must be at least 1 or greater.   plot(close,      "LW 1",   linewidth = 1)   plot(close + 5,  "LW 2",   linewidth = userWidth)   plot(close + 10, "LW 5",   linewidth = 5)   hline(240, "hline", color.maroon, linewidth = 3)   `

[Negative indices in arrays](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#negative-indices-in-arrays)
-----------------------------------------------------------------------------------------------------------------------------------------

Some array functions now accept negative indices.

In v5, array functions that require an element’s _index_ always expect a value _greater than or equal to 0_. Therefore, functions like [array.get()](https://www.tradingview.com/pine-script-reference/v6/#fun_array.get), [array.insert()](https://www.tradingview.com/pine-script-reference/v6/#fun_array.insert), [array.set()](https://www.tradingview.com/pine-script-reference/v6/#fun_array.set), and [array.remove()](https://www.tradingview.com/pine-script-reference/v6/#fun_array.remove) raise a runtime error if a negative index is passed.

In v6, [array.get()](https://www.tradingview.com/pine-script-reference/v6/#fun_array.get), [array.insert()](https://www.tradingview.com/pine-script-reference/v6/#fun_array.insert), [array.set()](https://www.tradingview.com/pine-script-reference/v6/#fun_array.set), and [array.remove()](https://www.tradingview.com/pine-script-reference/v6/#fun_array.remove) allow you to pass a _negative index_ to request items from the _end_ of the array. For example, `-1` refers to the last item in the array, `-2` refers to the second to last, and so forth.

``//@variable Array of "int" numbers from 1 to 5.   array<int> countingArray = array.from(1, 2, 3, 4, 5)   // Array indexing starts from 0 to retrieve first element in both v5 and v6.   int firstValue = countingArray.get(0)       //Returns "1"      // In v6, can retrieve last array element using negative index. This index is invalid in v5.   int lastValue = countingArray.get(-1)       //Returns "5"      // Other `array.*()` functions also accept negative indexing in v6. These lines raise runtime errors in v5.   countingArray.set(-2, 10)                   // Updated array: [1, 2, 3, 10, 5]   countingArray.remove(-5)                    // Updated array: [2, 3, 10, 5]   countingArray.insert(-1, 20)                // Updated array: [2, 3, 10, 20, 5]`` 

As a result, scripts that return a runtime error for using negative indices in v5 can be executed without error in v6.

However, if you create or update a script in v6, you must be aware of this new behavior to ensure that the script does not behave unexpectedly.

Keep in mind that negative indexing is still bound by the size of the array. Therefore, an array of 5 elements only accepts indexing from 0 to 4 (first to last element) or -1 to -5 (last to first element). Any other indices are out of bounds and raise a runtime error:

`//@variable Array of "int" numbers from 1 to 5.   array<int> countingArray = array.from(1, 2, 3, 4, 5)      // Trying to index negatively beyond the size of the array causes a runtime error.   countingArray.remove(-6)   `

[The \`transp\` parameter is removed](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#the-transp-parameter-is-removed)
-------------------------------------------------------------------------------------------------------------------------------------------------------

In Pine v4 and earlier, [plot()](https://www.tradingview.com/pine-script-reference/v6/#fun_plot) and similar functions had a `transp` parameter that specified the transparency of the resulting plot.

Pine v5 deprecated and hid the `transp` parameter, because it is not fully compatible with the color system that Pine currently uses. Using both transparency settings together can result in unexpected behavior, as the `transp` parameter can get overwritten by the transparency of the color passed to the function. In v5, using the [color.new()](https://www.tradingview.com/pine-script-reference/v6/#fun_color.new) function and not the `transp` parameter avoids any such conflicts.

Pine v6 removes the `transp` parameter completely from the following functions: [bgcolor()](https://www.tradingview.com/pine-script-reference/v6/#fun_bgcolor), [fill()](https://www.tradingview.com/pine-script-reference/v6/#fun_fill), [plot()](https://www.tradingview.com/pine-script-reference/v6/#fun_plot), [plotarrow()](https://www.tradingview.com/pine-script-reference/v6/#fun_plotarrow), [plotchar()](https://www.tradingview.com/pine-script-reference/v6/#fun_plotchar), and [plotshape()](https://www.tradingview.com/pine-script-reference/v6/#fun_plotshape). Whenever the [converter](https://www.tradingview.com/pine-script-docs/migration-guides/to-pine-version-6/#converting-v5-to-v6-using-the-pine-editor) encounters a `transp` argument, it removes the argument from the converted v6 script.

**Fix:** To set the transparency of a drawn plot, use the [color.new()](https://www.tradingview.com/pine-script-reference/v6/#fun_color.new) function. Pass the color value as the first argument, and the desired transparency value as the second.

For example, this v5 code uses the hidden `transp` parameter to set the color of the plot to `80` transparency:

`//@version=5   indicator("Transparency demo v5")      color myColor = close > open ? color.green : color.red   plot(close, color = myColor, transp = 80)   `

In Pine v6, the same result can be achieved using [color.new()](https://www.tradingview.com/pine-script-reference/v6/#fun_color.new):

`//@version=6   indicator("Transparency demo v6")      color myColor = close > open ? color.green : color.red   plot(close, color = color.new(myColor, 80))   `

If you need to preserve the color inputs in the “Settings/Style” menu, you must ensure that every color that gets passed to every [color.new()](https://www.tradingview.com/pine-script-reference/v6/#fun_color.new) call is qualified as either “const” or “input”. If at least one of these [color](https://www.tradingview.com/pine-script-reference/v6/#type_color) values is calculated dynamically (like the code above), the color selector does not appear in the settings:

![image](https://i3.wp.com/www.tradingview.com/pine-script-docs/_astro/MigrationGuideTov6-Transp-removed.UbKaK0oO_ZOtPJe.webp)

You can learn more about why this happens and how to avoid it [here](https://www.tradingview.com/pine-script-docs/concepts/colors/#maintaining-automatic-color-selectors).