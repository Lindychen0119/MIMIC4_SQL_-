# MIMIC4_SQL_-
# 1. 先從 services 撈出骨科病人 = ortho (N=26991)
create table `majestic-post-448508-p3.lindy_check_20250610.ortho` as select * FROM `majestic-post-448508-p3.mimic4_v3_1_hosp.services` 
where curr_service like "%ORTHO%"

# 2. ortho 去串 admissions = data (N=26991)
create table `majestic-post-448508-p3.lindy_check_20250610.data` as 
select A.*, B.* except(subject_id, hadm_id) 
from `majestic-post-448508-p3.lindy_check_20250610.ortho` as A
left join  `majestic-post-448508-p3.mimic4_v3_1_hosp.admissions` as B
on A.subject_id = B.subject_id and A.hadm_id = B.hadm_id;

# 3. data 去串 patients = data1 (N=26991)

create table `majestic-post-448508-p3.lindy_check_20250610.data1` as 
select A.*, B.* except(subject_id) 
from `majestic-post-448508-p3.lindy_check_20250610.data` as A
left join  `majestic-post-448508-p3.mimic4_v3_1_hosp.patients` as B
on A.subject_id = B.subject_id;

# 4. data1 去串 discharge = data2 (N=26991)
create table `majestic-post-448508-p3.lindy_check_20250610.data2` as 
select A.*, B.* except(subject_id, hadm_id) 
from `majestic-post-448508-p3.lindy_check_20250610.data1` as A
left join  `majestic-post-448508-p3.mimic4_v2_2_note.discharge` as B
on A.subject_id = B.subject_id and A.hadm_id = B.hadm_id;

# 5. 從 text 欄位中撈出title。
#### 該行結尾是冒號，冒號後沒東西的，逐行讀取，distinct後可以知道
CREATE OR REPLACE TABLE `majestic-post-448508-p3.lindy_check_20250610.data2_title` AS
SELECT
  *,
  (
    SELECT REGEXP_EXTRACT(TRIM(REGEXP_REPLACE(l, r'[：]', ':')), r'^(.*?):\s*$')
    FROM UNNEST(SPLIT(text, r'\r?\n')) AS l
    WHERE REGEXP_CONTAINS(TRIM(REGEXP_REPLACE(l, r'[：]', ':')), r'^.*:\s*$')
    LIMIT 1
  ) AS text_title
FROM `majestic-post-448508-p3.lindy_check_20250610.data2`;

# 上面都不會成功，直接換成python去做RE

# N=26991 中 discharge text有東西只有20058 筆資料
共10615 個
非 ___開頭 9578
___ 開頭 1037

### 20250611_title_extract.csv 是 majestic-post-448508-p3.lindy_check_20250610.data2











