# Evaluate a Manufacturing Process

### Project Overview

![manufacturing](https://github.com/user-attachments/assets/f5ca4a63-e82b-4709-83cb-833e12e833ab)

This project aims to improve manufacturing quality by applying Statistical Process Control (SPC). Using SQL, we analyze historical production data to calculate the Upper Control Limit (UCL) and Lower Control Limit (LCL) for product height. These limits define the acceptable range for production. Parts falling outside this range signal process issues that need correction. This data-driven approach helps maintain consistent product quality and ensures the manufacturing process runs efficiently with minimal defects.

<p>
UCL = avg_height + 3 × (stddev_height / √5)
</p>
<p>
LCL = avg_height − 3 × (stddev_height / √5)
</p>

The UCL defines the highest acceptable height for the parts, while the LCL defines the lowest acceptable height for the parts. Ideally, parts should fall between the two limits.

### Data Sources

The data is available in the `manufacturing_parts` table which has the following fields:
- `item_no`: the item number
- `length`: the length of the item made
- `width`: the width of the item made
- `height`: the height of the item made
- `operator`: the operating machine

### Exploratory Data Analysis (EDA)

EDA involved exploring the manufacturing_parts table to answer key questions, such as:
- How many parts are flagged as out of control (alert = TRUE)?
- Are certain operators generating more alerts (out-of-control parts) than others?
- Are there specific production periods (row_number ranges) where the process became unstable?

### Data Analysis

Including some interesting code/features worked with

```sql
-- Flag whether the height of a product is within the control limits
SELECT
	b.*,
	CASE
		WHEN 
			b.height NOT BETWEEN b.lcl AND b.ucl
		THEN TRUE
		ELSE FALSE
	END as alert
FROM (
	SELECT
		a.*, 
		a.avg_height + 3*a.stddev_height/SQRT(5) AS ucl, 
		a.avg_height - 3*a.stddev_height/SQRT(5) AS lcl  
	FROM (
		SELECT 
			operator,
			ROW_NUMBER() OVER w AS row_number, 
			height, 
			AVG(height) OVER w AS avg_height, 
			STDDEV(height) OVER w AS stddev_height
		FROM manufacturing_parts 
		WINDOW w AS (
			PARTITION BY operator 
			ORDER BY item_no 
			ROWS BETWEEN 4 PRECEDING AND CURRENT ROW
		)
	) AS a
	WHERE a.row_number >= 5
) AS b;
```

### Results/Findings

The result analysis are summarized as follows:
- Out of 363 total parts, 57 parts were flagged as out of control.
- Operator 5 has the highest number of alerts, indicating more frequent process deviations compared to others.
- Yes, for Operator 5 between row 15 and 17, the process shows frequent alerts and widening control limits, signaling instability.

### Recommendations

Based on the analysis, we recommend the following actions:
- Conduct a root cause analysis to investigate recurring process deviations.
- Provide additional training for Operator 5 on process control and standard operating procedures (SOP).
- Consider implementing pre-production checks or mid-shift inspections to catch similar issues early.

