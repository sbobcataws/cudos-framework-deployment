# Data Transfer Dashboard Enhancement Notes

## Overview
This document outlines the changes needed to add tags functionality and taxonomy explorer to the data-transfer dashboard, mirroring the functionality available in the cost-intelligence dashboard.

## Changes Needed for Data Transfer Dashboard

### 1. Add Tags Support to Data Transfer Dashboard

**Core Changes Required:**

#### A. Update Dashboard YAML Configuration
File: `dashboards/data-transfer/DataTransfer-Cost-Analysis-Dashboard.yaml`

1. **Add `tags_json` to nonTaxonomyColumns** (similar to cost-intelligence dashboard):
   ```yaml
   dashboards:
     DATATRANSFER COST ANALYSIS DASHBOARD ENHANCED:
       nonTaxonomyColumns:
       - product_code
       - service
       - operation
       - charge_type
       - usage_type
       - reservation_a_r_n
       - item_description
       - pricing_unit
       - region
       - pricing_term
       - linked_account_id
       - savings_plan_a_r_n
       - tags_json  # ADD THIS LINE
   ```

2. **Update view dependency** to include tags support:
   ```yaml
   views:
     data_transfer_view:
       dependsOn:
         tags: json  # ADD THIS LINE
         cur2:
           # existing dependencies...
   ```

#### B. Modify the data_transfer_view SQL Query
Update the SQL query in the view definition:

1. **Add tags_json to SELECT statement**:
   ```sql
   SELECT
     product_product_family product_family
   , product_servicecode
   , product['servicename'] product_servicename
   , line_item_product_code product_code
   , line_item_usage_start_date usage_date
   , bill_billing_period_start_date billing_period
   , bill_payer_account_id payer_account_id
   , line_item_usage_account_id linked_account_id
   , ${cur_tags_json} tags_json  -- ADD THIS LINE
   , product['product_name'] product_name
   -- ... rest of existing fields
   ```

2. **Update GROUP BY clause** to include tags_json:
   ```sql
   GROUP BY 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22  -- Add one more position
   ```

#### C. Update Dataset Definition
Add tags_json to the dataset InputColumns:

```yaml
InputColumns:
- Name: tags_json
  Type: STRING
# ... existing columns
```

And include it in ProjectedColumns:
```yaml
ProjectedColumns:
- tags_json
# ... existing columns
```

### 2. Add Taxonomy Explorer Sheet

**Requirements:**

#### A. Create Taxonomy Explorer Parameters
Add to the dashboard definition:

```yaml
Parameters:
- Name: TaxonomyExplorerGroupBy1
  Type: STRING
  DefaultValues:
    StaticValues:
    - Account
  AllowedValues:
    StaticValues:
    - Account
    # Add other taxonomy dimensions as needed
  
- Name: TaxonomyExplorerGroupBy2
  Type: STRING
  DefaultValues:
    StaticValues:
    - Account
  AllowedValues:
    StaticValues:
    - Account
    # Add other taxonomy dimensions as needed
```

#### B. Add Calculated Fields
Create calculated fields for taxonomy grouping:

```yaml
CalculatedFields:
- Expression: |-
    ifelse(
    ${TaxonomyExplorerGroupBy1} = 'Account', Account,
    // Add ${TaxonomyExplorerGroupBy1}
    Account
    )
  Name: Taxonomy Explorer Group By 1
  DataSetIdentifier: data_transfer_view

- Expression: |-
    ifelse(
    ${TaxonomyExplorerGroupBy2} = 'Account', Account,
    // Add ${TaxonomyExplorerGroupBy2}
    Account
    )
  Name: Taxonomy Explorer Group By 2
  DataSetIdentifier: data_transfer_view
```

#### C. Create Taxonomy Explorer Sheet
Add a new sheet with:
- Parameter controls for dimension selection
- Table visual showing cost breakdown
- Charts showing monthly trends by taxonomy dimensions
- Cost mapping visualization between dimensions

## Git Best Practices for Development

### Keeping Fork Up to Date

#### Initial Setup (One-time only):
```bash
# Add the original repository as upstream remote
git remote add upstream https://github.com/aws-samples/aws-cudos-framework-deployment.git

# Verify your remotes
git remote -v
```

#### Regular Sync Process:
```bash
# 1. Switch to your main branch
git checkout main

# 2. Fetch latest changes from upstream
git fetch upstream

# 3. Merge upstream changes into your main
git merge upstream/main

# 4. Push updated main to your fork
git push origin main

# 5. Switch to your feature branch (if exists)
git checkout feature/data-transfer-tags-taxonomy

# 6. Rebase your feature branch onto updated main
git rebase main

# 7. Force push your rebased feature branch (if already pushed)
git push --force-with-lease origin feature/data-transfer-tags-taxonomy
```

#### Quick Daily Sync:
```bash
# Morning routine - sync your fork
git checkout main
git pull upstream main
git push origin main
```

### Recommended Workflow:

1. **Create a feature branch**:
   ```bash
   git checkout -b feature/data-transfer-tags-taxonomy
   ```

2. **Make incremental commits**:
   ```bash
   # After each logical change
   git add .
   git commit -m "feat: add tags_json support to data-transfer view"
   git commit -m "feat: add taxonomy explorer to data-transfer dashboard"
   ```

3. **Keep commits focused and atomic** - each commit should represent one logical change

4. **Use conventional commit messages**:
   - `feat:` for new features
   - `fix:` for bug fixes
   - `docs:` for documentation
   - `test:` for tests

5. **Before submitting MR**:
   ```bash
   git rebase -i main  # Clean up commit history if needed
   git push origin feature/data-transfer-tags-taxonomy
   ```

## Setting Up Tests on Windows

### Test Environment Setup:

1. **Install Python dependencies**:
   ```cmd
   pip install -r requirements.txt
   pip install pytest  # For running Python tests
   ```

2. **Install BATS for shell script testing**:
   ```cmd
   # Install Git Bash or WSL for BATS support
   # Or use PowerShell equivalent testing
   ```

3. **Run Python tests**:
   ```cmd
   # From project root
   python -m pytest cid/test/python/ -v
   ```

4. **Run specific test files**:
   ```cmd
   python -m pytest cid/test/python/test_isolated_parameters.py -v
   ```

### Test Structure:
- **Python tests**: Located in `cid/test/python/`
- **BATS tests**: Located in `cid/test/bats/` (shell script tests)
- **CFN tests**: CloudFormation template tests in `cfn-templates/tests/`

### Running Tests Effectively:

1. **Set up virtual environment** (recommended):
   ```cmd
   python -m venv venv
   venv\Scripts\activate
   pip install -r requirements.txt
   pip install pytest
   ```

2. **Run tests with coverage**:
   ```cmd
   pip install pytest-cov
   python -m pytest cid/test/python/ --cov=cid --cov-report=html
   ```

3. **Test your changes**:
   - Create unit tests for any new functions you add
   - Test the dashboard deployment locally
   - Validate SQL queries against sample data

## Key Files to Modify

### Primary Files:
1. **`dashboards/data-transfer/DataTransfer-Cost-Analysis-Dashboard.yaml`** - Main dashboard config
2. **`cid/common.py`** - May need updates if data-transfer needs special tag handling

### Test Files:
3. **`cid/test/python/`** - Add test files for your new functionality

## Technical Implementation Details

### How Tags Work in the Current System:

The cid application uses a multi-step process to determine when to incorporate tags_json fields during dashboard deployment:

#### 1. View-Level Declaration
Views declare their need for tags in `resources.yaml` using:
```yaml
dependsOn:
  tags: json  # This signals the view needs tags_json functionality
```

#### 2. Dashboard-Level Configuration
Dashboards include `tags_json` in their `nonTaxonomyColumns` list:
```yaml
nonTaxonomyColumns:
  - product_code
  - service
  # ... other columns
  - tags_json  # This makes tags_json available as a filterable dimension
```

#### 3. Runtime Detection Logic
The key logic is in `common.py` around lines 1641-1654:

```python
# Check if any dependent views require tags_json
cur_tags_json_required = False
for dep_view_name in dataset_definition.get('dependsOn', {}).get('views', []):
    try:
        tags_type = self.resources['views'][dep_view_name]['dependsOn']['tags']
    except (KeyError, TypeError, AttributeError):
        tags_type = None
    
    if tags_type == 'json':  # This is the key check
        cur_tags_json_required = True
        break
```

#### 4. SQL Template Substitution
When `cur_tags_json_required = True`, the system:

1. **Generates the SQL**: Calls `generic_tags_json()` to create the complex SQL that converts tag columns into JSON format
2. **Template Replacement**: Replaces `${cur_tags_json}` placeholders in view SQL files with the generated SQL
3. **Custom Fields**: Creates parseJson expressions for individual tag access

#### 5. User Tag Selection
The `generic_tags_json()` function:
- Prompts users to select which tags they want (if not already specified)
- Generates SQL like:
```sql
json_format(
    CAST (
        MAP_FROM_ENTRIES (
            ARRAY[
                ('tag_environment', resource_tags_user_environment),
                ('tag_project', resource_tags_user_project)
            ]
        )
    AS JSON)
)
```

#### Key Decision Points
The system decides to include tags_json when:
1. **Any dependent view** has `dependsOn: tags: json`
2. **User has selected resource tags** during deployment
3. **Dashboard includes tags_json** in nonTaxonomyColumns

#### For Data Transfer Dashboard
The data transfer dashboard currently doesn't have tags support because:
1. Its views don't declare `dependsOn: tags: json`
2. The dashboard YAML doesn't include `tags_json` in nonTaxonomyColumns

To add tags support, you need to:
1. Add `tags: json` to the data transfer view's dependsOn section in `resources.yaml`
2. Include `tags_json` in the dashboard's nonTaxonomyColumns
3. Update the view SQL to include `${cur_tags_json}` in the SELECT statement

#### Legacy Implementation Details:
1. **Parameter Collection**: The `generic_tags_json()` function in `cid/common.py` handles tag parameter collection
2. **SQL Generation**: Tags are converted to JSON format using `json_format()` and `MAP_FROM_ENTRIES()`
3. **View Integration**: The `${cur_tags_json}` placeholder gets replaced with the generated SQL
4. **Dashboard Integration**: Tags appear as filterable dimensions in QuickSight

### Tags JSON SQL Pattern:
```sql
json_format(
    CAST (
        MAP_FROM_ENTRIES (
            ARRAY[
                ('tag_name1', resource_tags_tag_name1),
                ('tag_name2', resource_tags_tag_name2)
            ]
        )
    AS JSON)
)
```

### Taxonomy Explorer Pattern:
The taxonomy explorer uses dynamic calculated fields that switch between different grouping dimensions based on parameter values. This allows users to interactively explore cost data across different organizational dimensions.

## Notes and Considerations

1. **Performance Impact**: Adding tags can affect query performance. Choose only essential tags.

2. **Existing Patterns**: The tags functionality is well-established in the codebase - you'll be adapting existing patterns rather than creating new ones.

3. **Taxonomy Explorer Complexity**: The taxonomy explorer requires more work as it involves creating new dashboard sheets and visualizations.

4. **Testing Strategy**: Focus on testing the SQL query generation and parameter handling for tags functionality.

5. **Documentation**: Update any relevant documentation after implementing changes.


## Windows specific information

These are general notes about windows specific development needs.

- bats tooling for tests can be installed by using npm to deploy

```npn install -g bats```


## Deployment / Export testing

Notes from my conversation w/ Eric about making these changes and testing these changes.

### Testing Deployment

Testing deployment through different changes as I make them in my isengard accounts

Example:
```
cid-cmd deploy --dashboard-id media-services-insights  --athena-database customer_cur_data  --cur-table-name customer_all  --athena-workgroup primary --resources https://raw.githubusercontent.com/aws-solutions-library-samples/cloud-intelligence-dashboards-framework/refs/heads/media-dashboard-v1/dashboards/media-services-insights/msih.yaml
```

For my work on data transfer dashboard it would be:

```
cid-cmd deploy --dashboard-id datatransfer-cost-analysis-dashboard --athena-database cid2_data_export --cur-table-name cur2 --athena-workgroup CID --resources https://raw.githubusercontent.com/sbobcataws/cudos-framework-deployment/refs/heads/feature/data-transfer-tags-taxonomy/dashboards/data-transfer/DataTransfer-Cost-Analysis-Dashboard.yaml
```

```
cid-cmd delete --dashboard-id datatransfer-cost-analysis-dashboard 
```

### Exporting Dashboard for incorporating changes

Example:
```
cid-cmd export —analysis-id ec5ccb94-b6a2-4710-97d6-1e4d4eb92666 —dashboard-id media-services-insights —dashboard-export-method definition —export-known-datasets no —athena-database customer_cur_data —athena-workgroup primary —category Advanced —output msih.yaml —one-file yes
```

## References

- **Cost Intelligence Dashboard**: `dashboards/cost-intelligence/cost-intelligence.yaml` - Reference implementation for tags
- **CUDOS Dashboard**: `dashboards/cudos/CUDOS-v5-definition.yaml` - Reference implementation for taxonomy explorer
- **Common Functions**: `cid/common.py` - Core tag handling logic