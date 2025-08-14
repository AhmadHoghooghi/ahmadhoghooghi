You're right to notice that. The `agg` method's requirement for a `TypedColumn` is a key design choice that, while most powerfully used with a custom `Aggregator`, isn't limited to it.

The other main option is to use Spark's **built-in aggregate functions** and make them type-safe by explicitly providing an **encoder**.

---

### ## The Two Sources for a `TypedColumn`

You can generate the `TypedColumn` needed for `KeyValueGroupedDataset.agg()` in two primary ways:

#### 1. Custom `Aggregator`

This is the approach for creating completely new, complex aggregation logic. You define the entire process (`zero`, `reduce`, `merge`, `finish`) from scratch.

- **How:** You create an instance of your custom `Aggregator` and call `.toColumn()` on it.
    
- **Use Case:** Calculating a geometric mean, a weighted average, or aggregating complex objects into a summary object—logic that doesn't exist in Spark's standard library.
    

#### 2. Built-in Functions + Encoder

This is the approach for using Spark's standard, highly-optimized aggregate functions (like `sum`, `avg`, `count`, etc.) within the type-safe API.

- **How:** The functions in `org.apache.spark.sql.functions` (e.g., `sum()`, `avg()`) normally return a generic `Column`. To make them type-safe, you call the **`.as(Encoder)`** method on them. This tells Spark, "Use your built-in function, and I guarantee the result can be encoded as this specific type."
    
- **Use Case:** You need a standard aggregation but are already working within the `groupByKey` paradigm. This gives you the performance of Spark's native functions combined with the type-safety of Datasets.
    

---

### ## Practical Java Example: Combining Both

The real power becomes apparent when you combine both approaches in a single, efficient, multi-aggregation call. Let's find the total number of sales (with a custom `Aggregator`) and the total revenue (with the built-in `sum` function) for each category.

Java

```
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Encoders;
import org.apache.spark.sql.TypedColumn;
import scala.Tuple3;
import static org.apache.spark.sql.functions.sum;

// Assume 'salesDS' is a Dataset<Sale> and 'CountAggregator' exists from the previous example
CountAggregator countAgg = new CountAggregator();

// --- Create TypedColumns from both sources ---

// 1. From our custom Aggregator
TypedColumn<Sale, Long> countCol = countAgg.toColumn().name("number_of_sales");

// 2. From a built-in function + Encoder
//    We are telling Spark that the result of sum("price") is a Double.
TypedColumn<Sale, Double> revenueCol = sum(salesDS.col("price"))
                                           .as(Encoders.DOUBLE())
                                           .name("total_revenue");


// --- Use both in a single, efficient .agg() call ---
Dataset<Tuple3<String, Long, Double>> results = salesDS
    .groupByKey(Sale::getCategory, Encoders.STRING())
    .agg(countCol, revenueCol);

results.show();
// +-----------+-----------------+---------------+
// |         _1|number_of_sales|  total_revenue|
// +-----------+-----------------+---------------+
// |Electronics|              150|       53392.50|
// |   Clothing|              320|       14576.00|
// +-----------+-----------------+---------------+
```

---

### ## When to Use Which

|Scenario|Recommended Approach|Reason|
|---|---|---|
|You need a **standard aggregation** like sum, average, count, max, or min.|Use a **built-in function + `.as(Encoder)`**.|It's simpler to write and uses Spark's native, highly-optimized implementations.|
|Your aggregation logic is **complex, custom, or involves manipulating whole objects**.|Create a custom **`Aggregator`**.|It provides complete flexibility and compile-time type-safety for any logic you can imagine.|