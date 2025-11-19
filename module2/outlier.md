# ✅ **Outlier Detection Algorithms (IQR & Z-Score) — with Full Explanation**

---

# ## **1. IQR Method (Interquartile Range)**

### **Python implementation**

```python
def detect_outliers_iqr(data):
    # Step 1: Sort the data
    data_sorted = sorted(data)
    
    n = len(data_sorted)

    # Step 2: Calculate Q1 and Q3 positions
    pos_q1 = 0.25 * (n + 1)
    pos_q3 = 0.75 * (n + 1)

    # Helper function: get quartile using interpolation
    def get_quartile(pos):
        lower_index = int(pos) - 1
        fractional = pos - int(pos)

        # If position is exactly an integer index
        if fractional == 0:
            return data_sorted[lower_index]
        
        # If the position falls between two values → interpolate
        next_index = lower_index + 1
        return data_sorted[lower_index] + fractional * (data_sorted[next_index] - data_sorted[lower_index])

    # Step 3: Compute Q1 and Q3 using the helper
    Q1 = get_quartile(pos_q1)
    Q3 = get_quartile(pos_q3)

    # Step 4: Compute the IQR
    IQR = Q3 - Q1

    # Step 5: Determine the outlier boundaries
    lower_bound = Q1 - 1.5 * IQR
    upper_bound = Q3 + 1.5 * IQR

    # Step 6: Identify the outliers
    outliers = [x for x in data if x < lower_bound or x > upper_bound]

    return {
        "Q1": Q1,
        "Q3": Q3,
        "IQR": IQR,
        "lower_bound": lower_bound,
        "upper_bound": upper_bound,
        "outliers": outliers
    }

# Example usage:
data = [30, 25, 32, 40, 22, 35, 28, 20, 7, 75]
result = detect_outliers_iqr(data)
print(result)
```

---

## **Explanation of Each Variable (IQR Method)**

| Variable            | Meaning                                          | Purpose                                                            |
| ------------------- | ------------------------------------------------ | ------------------------------------------------------------------ |
| `data_sorted`       | The sorted dataset                               | Quartiles must be computed on sorted data                          |
| `n`                 | Number of values in dataset                      | Used to compute quartile positions                                 |
| `pos_q1`            | The theoretical position of Q1 (25th percentile) | Used to locate Q1 between sorted values                            |
| `pos_q3`            | The theoretical position of Q3 (75th percentile) | Used to locate Q3 between sorted values                            |
| `get_quartile(pos)` | Helper function that uses interpolation          | Accurately calculates Q1 and Q3 even if position is not an integer |
| `Q1`                | First quartile (25%)                             | Lower boundary of the “middle 50%” of data                         |
| `Q3`                | Third quartile (75%)                             | Upper boundary of the “middle 50%” of data                         |
| `IQR`               | `Q3 - Q1`                                        | Spread of the middle 50% of data                                   |
| `lower_bound`       | `Q1 - 1.5 × IQR`                                 | Anything below = low outlier                                       |
| `upper_bound`       | `Q3 + 1.5 × IQR`                                 | Anything above = high outlier                                      |
| `outliers`          | List of outlier values                           | Final result showing extreme values                                |

---

# ## **2. Z-Score Outlier Detection Method**

### **Python implementation**

```python
def detect_outliers_zscore(data):
    n = len(data)

    # Step 1: Compute the mean (average)
    mean_value = sum(data) / n

    # Step 2: Compute the standard deviation
    # 2a: Compute squared deviations
    squared_deviations = [(x - mean_value) ** 2 for x in data]

    # 2b: Compute variance
    variance = sum(squared_deviations) / n

    # 2c: Standard deviation is sqrt of variance
    std_dev = variance ** 0.5

    # Step 3: Compute Z-scores for each value
    z_scores = [(x - mean_value) / std_dev for x in data]

    # Step 4: Identify outliers (|z| > 3)
    outliers = [data[i] for i in range(n) if abs(z_scores[i]) > 3]

    return {
        "mean": mean_value,
        "std_dev": std_dev,
        "z_scores": z_scores,
        "outliers": outliers
    }

# Example usage:
data = [30, 25, 32, 40, 22, 35, 28, 20, 7, 75]
result = detect_outliers_zscore(data)
print(result)
```

---

## **Explanation of Each Variable (Z-Score Method)**

| Variable             | Meaning                                     | Purpose                                                                |
| -------------------- | ------------------------------------------- | ---------------------------------------------------------------------- |
| `mean_value`         | Average of the dataset                      | Center of the distribution                                             |
| `squared_deviations` | Each value’s squared distance from the mean | Used to compute variance                                               |
| `variance`           | Average squared deviation                   | Measure of spread                                                      |
| `std_dev`            | Standard deviation (√variance)              | Average distance from the mean                                         |
| `z_scores`           | `(x - mean) / std_dev` for each value       | Measures how many standard deviations away from the mean each value is |
| `outliers`           | Values where `abs(z) > 3`                   | Final detected outliers                                                |

---

# ## **3. Summary Table: IQR vs Z-Score Algorithms**

| Step                 | IQR Method                           | Z-Score Method            |   |     |
| -------------------- | ------------------------------------ | ------------------------- | - | --- |
| **Sorting required** | ✔ Yes                                | ✖ No                      |   |     |
| **Uses mean**        | ✖ No                                 | ✔ Yes                     |   |     |
| **Uses std dev**     | ✖ No                                 | ✔ Yes                     |   |     |
| **Key statistic**    | Q1, Q3, IQR                          | Mean, Standard Deviation  |   |     |
| **Outlier Rule**     | x < Q1 − 1.5×IQR or x > Q3 + 1.5×IQR |                           | z | > 3 |
| **Best for**         | Skewed or non-normal data            | Normally-distributed data |   |     |

---


