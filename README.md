# Comorbidity Score Calculator
## Introduction
Comorbidity scores are a tool used in healthcare and research to measure and quantify the burden of diseases in a patient. There are many different scoring systems present, some of which gained popularity and are widely used as standardized tools to quantify the impact of the comorbidities on outcomes such as mortality or occurence of complications. 

This project aims to be a tool for calculating a comorbidity score automatically when given an input consisting of ICD-10 codes.

For now, only **Charlson Comorbidity Index** (Deyo's modification) is available. In future, additional scoring systems might be added to the project's library.

The Charlson Comorbidity Index was first developed with an intention to stratify the patient risk groups in clinical trials to better predict the impact of the studied disease and therapy on the patients mortailty.

Over the years there have been many iterations and modifications of the original CCI. One that gained popularity is a midification by Deyo and it is the one used in this tool.

The categories in the Deyo's modification of the Charlson Comorbidity Index and the points they score are as follows:

| Category                                   | Score |
|--------------------------------------------|-------|
| Myocardial infarction                      | 1     |
| Congestive heart failure                   | 1     |
| Peripheral vascular disease                | 1     |
| Cerebrovascular disease                    | 1     |
| Dementia                                   | 1     |
| Chronic pulmonary disease                  | 1     |
| Rheumatologic disease                      | 1     |
| Peptic ulcer disease                       | 1     |
| Mild liver disease                         | 1     |
| Diabetes                                   | 1     |
| Diabetes with chronic complications        | 2     |
| Hemiplegia or paraplegia                   | 2     |
| Renal disease                              | 2     |
| Any malignancy including leukemia and lymphoma | 2 |
| Moderate or severe liver disease           | 3     |
| Metastatic solid tumor                     | 6     |
| HIV/AIDS                                   | 6     |



## Usage
### Specify the JSON file's path
Within the 'calc.py' file, update the location of the JSON file locally for the function to work properly
### The main function
The main function of the project, calculate_score(), takes in 3 arguments, all of which are keyword arguments, meaning they must be explicitly named when calling the function:
- icd_codes - a string or a list of strings with the ICD-10 codes from the patients' diagnoses
- mapping - a string, which mapping to use for the calculation
- exact_code - a boolean

#### icd_codes
The ICD-10 codes, meaning diagnoses of a single patient. These are used to decide which of categories mentioned in the Introduction score points.
#### mapping
Identifier for the version of the ICD code mapping to be used. Valid options for the current version inculde:
- "cci_icd2024gm" : the 2024 version of the German Modification ICD-10 codes, mapped by the algorithm authors
- "cci_icd2024gm_quan": a variation based on Quan's implementation, applied to the 2024 ICD-10 GM codes.
- "cci_icd2024_quan_orig" : Quan's mapping, as presented and explained in the following paper , DOI: 10.1097/01.mlr.0000182534.19832.83


This argument is optional, by default it is set to "cci_icd2024gm".
#### exact_codes
If True, checks for exact matches between ICD codes and the mapping data. If False, checks for prefix matches. Default is False, meaning that if any of the codes in the input start with any of the codes in the selected mapping list, it scores.
Example: In the selected mapping, there is a code 'K70.48' for 'Other unspecified alcoholic liver failure'. The code is listed under the category 'severe liver disease' and scores 3 points. If in the input a code 'K70.4811' is provided, it still scores 3 points, as it starts with 'K70.48'. This was implemented as a means of ensuring that if any of the codes is extended with subcodes, they all still score points even if not all are yet implemented in the package's library.

### Output
The output is a tuple.
The first element is an integer, ranging from 0 to 29, and meaning the total scored points. 
The second element is a list of the categories that scored points when calculating the score.

## Mapping logic
The mapping is stored in a JSON file. It organizes ICD-10 codes into categories representing different comorbidities (see Table in the Introduction section). Each category has attributes such as:
- condition: ('any' or 'both') Specifies if scoring should occur when any code from the list matches, or if both subgroups of codes must match (useful for complex conditions with multiple subgroups).
- codes: A list of ICD codes or groups of codes. Each code (or group) can either match exactly or by prefix, depending on the function’s input parameter (see Usage)
- weight: Points scored by the category. Each category has a weight that adds to the total score if the condition is met.
- depends_on: Specifies dependencies among categories, ensuring only the more severe condition scores if both are present (e.g., severe liver disease supersedes mild liver disease).

Example structure:
{
    "cci_icd2024gm": {
        "myocardial_infarction": {
            "condition": "any",
            "codes": ["I21", "I22"],
            "weight": 1
        },
        "diabetes": {
            "condition": "any",
            "codes": ["E10", "E11"],
            "weight": 1,
            "depends_on": ["diabetes_complicated"]
        },
        "diabetes_complicated": {
            "condition": "any",
            "codes": ["E13"],
            "weight": 2
        }
    }
}

By implementing this structure it was possible to make the package flexible in terms of updating the mapping data. Adding, extending, or modifying the current mappings can be done by changing only the JSON file and without changing the core code.


## About current mappings
- "cci_icd2024_quan_orig" : Quan's original mapping, as presented and explained in the following paper , DOI: 10.1097/01.mlr.0000182534.19832.83
- "cci_icd2024gm" : the 2024 version of the German Modification ICD-10 codes, mapped by the algorithm authors. It was prepared by two authors independently assigning ICD-10 codes from the 2024 German Modification list to each of the categories. In case when inconsistencies were present, these were discussed until a consensus was found.
- "cci_icd2024gm_quan": a variation based on Quan's implementation, applied to the 2024 ICD-10 GM codes. It was prepared by comparing the Quan's list of ICD-10 codes with the 2024 German Modification list (2024-GM-ICD-10), removing the codes that were no longer present and, if neccessary, expanding the codes that had been expanded, so that all the subcodes are present in the list.

A table in .md format with all ICD-10 codes is available in the package's root directory.

## License

This project is licensed under the MIT License, a permissive open-source license that allows for reuse, modification, and distribution. Under the MIT License, you are free to use this software in both personal and commercial projects, provided that the original copyright notice and license terms are included in any copies or substantial portions of the software.
The license text is available as a .txt file in the package's root directory.

# Mapping Table
The following is a mapping table showing exactly ICD-10 codes for each mapping

|                                                 | The authors' ICD-2024-GM mapping                             | Quan's mapping, adjusted for ICD-2024-GM                     | Quan's original mapping                                      |
| ----------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Liver disease, moderate to severe               | I98.2 AND (K70.x OR K71.x OR K74.x), I98.3 AND (K70.x OR K71.x OR K74.x), K70.4x, K71.1, K72.7x,  K72.x, K74.71, K74.72, K76.5-K76.7 | I85.0,  I85.9, I86.4, I98.2, K70.4x, K71.1, K72.1x, K72.9, K76.5, K76.6, K76.7 | I85.0,  I85.9, I86.4, I98.2, K70.4, K71.1, K72.1, K72.9, K76.5, K76.6, K76.7 |
| Liver disease, mild                             | B18.x, K70.x, K71.x (except for K71.1), K73.x, K74.x (except for K74.71 and K74.72), K76.x (except for K76.5-K76.7), K77.x, Z94.4 | B18.x, K70.0-K70.3, K70.9, K71.3-K71.5, K71.7, K73.x, K74.x, K76.0, K76.2-K76.4,  K76.8, K76.9, Z94.4 | B18.x,  K70.0-K70.3, K70.9, K71.3-K71.5, K71.7, K73.x, K74.x, K76.0, K76.2-K76.4,  K76.8, K76.9, Z94.4 |
| Malignancy, not metastatic                      | C00.x-C26.x, C30.x-C34.x, C37.x- C41.x, C43.x, C45.x-C58.x, C60.x- C76.x, C81.x-C85.x, C88.x, C90.x-C97.x | C00.x-C26.x,  C30.x-C34.x, C37.x- C41.x, C43.x, C45.x-C58.x, C60.x- C76.x, C81.x-C85.x,  C88.x, C90.x-C97.x | C00.x-C26.x,  C30.x-C34.x, C37.x- C41.x, C43.x, C45.x-C58.x, C60.x- C76.x, C81.x-C85.x,  C88.x, C90.x-C97.x |
| Malignancy , metastatic                         | C77.x-C80.x                                                  | C77.x-C80.x                                                  | C77.x-C80.x                                                  |
| AIDS                                            | B20-B22, B24, U60.3                                          | B20-B22, B24                                                 | B20.x-B22.x,  B24.x                                          |
| Cerebrovascular                                 | G45.x, G46.x, H34.0, I60.x-I69.x                             | G45.x, G46.x, H34.0, I60.x-I69.x                             | G45.x,  G46.x, H34.0, I60.x-I69.x                            |
| Chronic pulmonary disease                       | I27.8, I27.9, J40.x-J47.x, J60.x-J67.x, J68.4, J70.1, J70.3, J84.1x, J96.1, J99.2x, T86.81 | I27.8,  I27.9, J40.x-J47.x, J60.x-J67.x, J68.4, J70.1, J70.3 | I27.8,  I27.9, J40.x-J47.x, J60.x-J67.x, J68.4, J70.1, J70.3 |
| Rheumatic disease                               | M05.x, M06.x, M31.5, M32.x-M34.x, M35.1, M35.3, M36.0        | M05.x,  M06.x, M31.5, M32.x-M34.x, M35.1, M35.3, M36.0       | M05.x,  M06.x, M31.5, M32.x-M34.x, M35.1, M35.3, M36.0       |
| Dementia                                        | F00.x-F03.x, F05.1, G30.x, G31.0, G31.1, U63.x               | F00.x-F02.x, F03,  F05.1, G30.x, G31.1                       | F00.x-F03.x,  F05.1, G30.x, G31.1                            |
| Diabetes mellitus with chronic complications    | E10.2-E10.5, E10.7, E11.2-E11.5, E11.7, E12.2-E12.5, E12.7, E13.2- E13.5, E13.7, E14.2-E14.5, E14.7 | E10.2-E10.5,  E10.7, E11.2-E11.5, E11.7, E12.2-E12.5, E12.7, E13.2- E13.5, E13.7,  E14.2-E14.5, E14.7 | E10.2-E10.5,  E10.7, E11.2-E11.5, E11.7, E12.2-E12.5, E12.7, E13.2- E13.5, E13.7,  E14.2-E14.5, E14.7 |
| Diabetes mellitus without chronic complications | E10.0, E10.1, E10.6, E10.8, E10.9, E11.0, E11.1, E11.6, E11.8, E11.9, E12.0, E12.1, E12.6, E12.8, E12.9, E13.0, E13.1, E13.6, E13.8, E13.9, E14.0, E14.1, E14.6, E14.8, E14.9 | E10.0, E10.1, E10.6, E10.8, E10.9, E11.0, E11.1, E11.6, E11.8, E11.9, E12.0, E12.1, E12.6, E12.8, E12.9, E13.0, E13.1, E13.6, E13.8, E13.9, E14.0, E14.1, E14.6, E14.8, E14.9 | E10.0,  E10.1, E10.6, E10.8, E10.9, E11.0, E11.1, E11.6, E11.8, E11.9, E12.0, E12.1,  E12.6, E12.8, E12.9, E13.0, E13.1, E13.6, E13.8, E13.9, E14.0, E14.1, E14.6,  E14.8, E14.9 |
| Heart failure                                   | I09.9, I11.0x, I13.0, I13.2, I25.5, I42.x, I43.x, I50.x, P29.0 | I09.9,  I11.0x, I13.0x, I13.2x, I25.5, I42.0, I42.5-I42.9, I43.x, I50.x, P29.0 | I09.9,  I11.0, I13.0, I13.2, I25.5, I42.0, I42.5-I42.9, I43.x, I50.x, P29.0 |
| Kidney disease                                  | I12.0x, I13.1, I13.2, N03.2-N03.7, N05.2-N05.7, N18.x, N19, N25.0, Z49.1, Z49.2, Z94.0, Z99.2 | I12.0x,  I13.1x, N03.2-N03.7, N05.2- N05.7, N18.x, N19, N25.0, Z49.0- Z49.2, Z94.0,  Z99.2 | I12.0,  I13.1, N03.2-N03.7, N05.2- N05.7, N18.x, N19.x, N25.0, Z49.0- Z49.2, Z94.0,  Z99.2 |
| Myocardial Infarction                           | I21.x, I22.x, I25.2x                                         | I21.x,  I22.x, I25.2x                                        | I21.x,  I22.x, I25.2                                         |
| Paraplegia and hemiplegia                       | G04.1, G11.4, G80.0-G80.2, G81.x, G82.x, G83.0, G83.5, G83.9 | G04.1,  G11.4, G80.1, G80.2, G81.x, G82.x, G83.0-G83.4, G83.9 | G04.1,  G11.4, G80.1, G80.2, G81.x, G82.x, G83.0-G83.4, G83.9 |
| Peptic ulcer                                    | K25.x-K28.x                                                  | K25.x-K28.x                                                  | K25.x-K28.x                                                  |
| Peripheral vascular disease                     | I70.x, I71.x, I73.1, I73.8, I77.1, I79.2, K55.1, K55.9, Z95.88, Z95.9 | I70.x, I71.x, I73.1, I73.8, I73.9, I77.1, I79.0, I79.2, K55.1, K55.8, K55.9, Z95.8x, Z95.9 | I70.x, I71.x, I73.1, I73.8, I73.9, I77.1, I79.0, I79.2, K55.1, K55.8, K55.9, Z95.8, Z95.9 |
