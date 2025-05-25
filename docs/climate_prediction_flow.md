# Climate prediction script overview

This document explains the workflow implemented in `_main_make_predictions_for_climate_p.py`. The script trains several DeepVARwT models and generates forecasts for global temperatures.

## Workflow summary

1. **Load dataset and configuration**: read `temperature_data.csv` and define model parameters such as the VAR order, number of layers and training iterations.
2. **Forecasting loop**: for each forecast origin (20 sections by default) the `process_section` function is executed.
3. **Model selection inside each section**:
   - Iterate over different numbers of time functions (`num_of_t`) and hidden dimensions (`num_of_h`).
   - For each combination call `train_network` from `_model_fitting_for_real_data.py` to fit the DeepVARwT model.
   - Log‑likelihood values are stored to determine the best model.
4. **Forecast generation**: the chosen model is passed to `forecast_based_on_pretrained_model` which
   - loads the trained parameters,
   - computes one‑step ahead trend values,
   - constructs VAR coefficients and predicts future observations,
   - calculates prediction intervals and accuracy metrics (MAPE and MSIS).
   - Forecasts and residuals are written to CSV files for later inspection.
5. **Result aggregation**: after all sections finish, averaged MAPE and MSIS values are printed.

## Mermaid overview

```mermaid
flowchart TD
    start([Start]) --> init["Load data & set parameters"]
    init --> parallel["Parallel loop over sections"]
    parallel --> proc["process_section"]
    proc --> grid["Loop over num_of_t & num_of_h"]
    grid --> train["train_network"]
    train --> grid
    grid --> select["Choose lowest loss model"]
    select --> predict["forecast_based_on_pretrained_model"]
    predict --> save["Save forecasts"]
    save --> ret["Return metrics"]
    ret --> parallel
    parallel --> aggr["Aggregate metrics"]
    aggr --> end([End])
```

The diagram highlights the main steps and their relations for easier navigation through the code.
