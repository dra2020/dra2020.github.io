# Dataset Key Builder

A Python script to parse natural language input and generate dataset keys for redistricting and election analysis.

## Usage

### Command Line Mode
```bash
python3 dataset_keys.py "2024 president"
# Output: E24GPR

python3 dataset_keys.py "2020 demo vap nh"
# Output: D20TNH
```

### Interactive Mode
```bash
python3 dataset_keys.py
```

Then enter specifications at the prompt. Keys are accumulated and displayed as a space-separated list:
```
Enter dataset: 2024 president
"E24GPR"

Enter dataset: 2020 demo
"E24GPR D20T"

Enter dataset: 2020 demo nh
"E24GPR D20T D20TNH"

Enter dataset: quit

Final list:
"E24GPR D20T D20TNH"
Goodbye!
```

## Input Format

The script accepts flexible, natural language input with these components:

### Election Keys

#### Required
- **Year**: 4-digit (2024) or 2-digit (24)

#### Optional (with defaults)
- **Type**: election (default), composite, pvi
- **Subtype**: general (default), primary, special
- **Office**: president, senate, governor, ag, etc.
- **Suffix**: runoff, special, or digit 0-9

### Demographic Keys

#### Required
- **Year**: 4-digit (2024) or 2-digit (24)
- **Type**: demographic, demo

#### Optional (with defaults)
- **Voting Age**: voting/vap (default), total/all
- **Suffix**: nh/non-hispanic, acs/survey, adjusted

### Features
- **Case insensitive**: "President", "president", "PRESIDENT" all work
- **Partial matching**: "pres", "sen", "gov", "demo" are recognized
- **Flexible order**: Components can appear in any order

## Examples

### Election Examples

| Input | Output | Description |
|-------|--------|-------------|
| `2024 president` | `E24GPR` | 2024 General Presidential Election |
| `2024 Sen` | `E24GSE` | 2024 General Senate Election |
| `2024 composite` | `C24GCO` | 2024 General Composite |
| `22 governor` | `E22GGO` | 2022 General Gubernatorial Election |
| `2020 pres runoff` | `E20GPRR` | 2020 General Presidential Runoff |
| `2021 senate special` | `E21SSE` | 2021 Special Senate Election |

### Demographic Examples

| Input | Output | Description |
|-------|--------|-------------|
| `2020 demo` | `D20T` | 2020 Demographic - Voting Age Population |
| `2020 demographic voting` | `D20T` | 2020 Demographic - Voting Age Population |
| `2020 demo total` | `D20F` | 2020 Demographic - Total Population |
| `2020 demo vap nh` | `D20TNH` | 2020 Demographic - VAP, Non-Hispanic |
| `2020 demo acs` | `D20TACS` | 2020 Demographic - VAP, ACS |
| `2020 demo adjusted` | `D20TA` | 2020 Demographic - VAP, Adjusted |
| `2020 demo total non-hispanic` | `D20FNH` | 2020 Demographic - Total Pop, Non-Hispanic |
| `20 demographic all ages survey` | `D20FACS` | 2020 Demographic - Total Pop, ACS |

## Office Codes

| Code | Office |
|------|--------|
| PR | President |
| SE | Senate |
| GO | Governor |
| AG | Attorney General |
| CO | Composite/Congress |
| LG | Lieutenant Governor |
| TR | Treasurer |
| CP | Comptroller |
| AU | Auditor |
| SS | Secretary of State |
| SC | State Supreme Court |
| IX | Index |

## Demographic Components

### Voting Age
- **T** (True): Voting age population only (default)
  - Keywords: voting, vap, adult
- **F** (False): Total population (all ages)
  - Keywords: total, all, allages, population

### Demographic Suffixes
- **NH**: Non-Hispanic population
  - Keywords: nh, non-hispanic, nonhispanic
- **ACS**: American Community Survey data
  - Keywords: acs, survey
- **A**: Adjusted for prisoner population
  - Keywords: adjusted, adj, prisoner

## Dataset Key Format

### Election Keys
Format: `[Type][Year][Subtype][Office][Suffix]`
- Example: `E24GPR` = Election, 2024, General, President

### Demographic Keys
Format: `[Type][Year][VotingAge][Suffix]`
- Example: `D20TNH` = Demographic, 2020, Voting Age, Non-Hispanic

## Requirements

- Python 3.6+
- `dataset_key.json` in the same directory or `scripts/` subdirectory
