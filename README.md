# การวิเคราะห์แคมเปญการตลาดของธนาคาร

ในบทความนี้ เราจะมาวิเคราะห์ชุดข้อมูลจากแคมเปญการตลาดโดยตรงของธนาคารโปรตุเกสที่ใช้การโทรศัพท์เพื่อโปรโมทการสมัครสมาชิกเงินฝากธนาคาร โดยมีเป้าหมายหลักเพื่อหาปัจจัยที่มีผลต่อความสำเร็จของแคมเปญหรือการสมัครสมาชิก (ตัวแปร y) ว่ามีตัวแปรใดบ้างที่สามารถทำนายหรืออธิบายได้ถึงการตอบรับของลูกค้า นอกจากนี้เรายังจะใช้เทคนิคต่างๆ ในการวิเคราะห์ข้อมูล เช่น การสร้างโมเดลการถดถอย (Logistic Regression) เพื่อให้เห็นภาพรวมของปัจจัยที่มีผลต่อผลลัพธ์ โดยในตัวอย่างต่อไปนี้ เราจะใช้โปรแกรมภาษา R ในการเขียน code เพื่อทำนายผลค่ะ

## Download Dataset

การดาวน์โหลดและโหลดข้อมูลใน R สามารถทำได้ตามขั้นตอนดังนี้:

1. ใช้ package `tidyverse` ที่มีฟังก์ชันสำหรับจัดการข้อมูล
2. ใช้ฟังก์ชัน read.delim() หรือ readr::read_csv() เพื่อโหลดไฟล์ CSV โดยตั้งค่าตัวแยก (delimiter) เป็น semi-colon ได้ดังนี้:

```r
suppressPackageStartupMessages(library(tidyverse)) 
bank = read_delim('data/bank-marketing.csv', delim=";", show_col_types = FALSE)
nrow(bank)
head(bank, n = 100)
```
## **Clean Data**
ก่อนที่เราจะเริ่มการวิเคราะห์ข้อมูลจากชุดข้อมูลนี้ เราจำเป็นต้องทำความสะอาดข้อมูลเบื้องต้นก่อน เพื่อให้มั่นใจว่าไม่มีข้อมูลที่ไม่จำเป็นหรือผิดปกติส่งผลกระทบต่อผลลัพธ์ที่ได้

## **ตรวจสอบข้อมูลที่ซ้ำกัน**
```r
# cleaning data set
# check duplicate value by columns
bank_job <- bank %>% distinct(job)
bank_marital <- bank %>% distinct(marital)
bank_edu <- bank %>% distinct(education)
bank_defu <- bank %>% distinct(default)
bank_housing <- bank %>% distinct(housing)
bank_loan <- bank %>% distinct(loan)
bank_contact <- bank %>% distinct(contact)
bank_month <- bank %>% distinct(month)
bank_day <-bank %>% distinct(day_of_week)
bank_poutcome <-bank%>% distinct(poutcome)
bank_y <-bank %>% distinct(y)

```
## **แปลงค่า "unknown" เป็น null**
```r
#convert unknown value to null for easy to handle
bank <- bank %>%
  mutate(across(c(job, marital, education, default, housing, loan), as.character)) %>%
  mutate(across(c(job, marital, education, default, housing, loan), ~ na_if(., "unknown")))
```

## **ลบแถวที่มีค่า null**
```r
bank_cleaned <- bank %>% na.omit() # remove null value in rows
```

## **เช็คประเภทข้อมูลแต่ละคอลัมน์**
```r
str(bank_cleaned)
```

## แปลงข้อมูล
```r
# convert all charactors to factors
bank_cleaned[] <- lapply(bank_cleaned, function(x) if(is.character(x)) factor(x) else x)
```

## **กำหนด Label ให้กับตัวแปร Objective (y)**
```r
bank_cleaned$y <- factor(bank_cleaned$y, levels = c("no", "yes"))

```
## **ใช้ Model ที่เหมาะสมในการทำนาย**
ในที่นี้ เราใช้ Logistic Regression เพื่อวิเคราะห์ความสัมพันธ์ระหว่างตัวแปรอิสระ (Independent Variables) และตัวแปรเป้าหมาย (y) ซึ่งเป็น ข้อมูลประเภท Binary (yes/no)
```r
model <- glm(y ~ ., family = binomial, data = bank_cleaned)
summary(model)

```

## **อ่านผลลัพธ์จากการรันโมเดล Logistic Regression**
### ตัวแปรที่มีนัยสำคัญทางสถิติ (p-value < 0.05)
#### ข้อมูลลูกค้า
jobblue-collar (p = 0.0196)
jobretired (p = 0.00525)
jobstudent (p = 0.0356)
educationuniversity.degree (p = 0.0269)
### ข้อมูลการติดต่อ
contacttelephone (p < 0.0001)
duration (p < 0.0001)
campaign (p = 0.0019)
pdays (p = 0.0000343)
#### ปัจจัยทางเศรษฐกิจ
emp.var.rate (p < 0.0001)
cons.price.idx (p < 0.0001)
euribor3m (p = 0.0487)
#### ตัวแปรที่ไม่มีนัยสำคัญ (p > 0.05)
age (p = 0.419)
marital status (p > 0.05)
defaultyes (p = 0.9487)
housing loan & personal loan (p > 0.05)
#### การประเมินคุณภาพของโมเดล
Null deviance: 23160
Residual deviance: 13902
AIC: 13998

## ผลลัพธ์ที่ได้ ตอบโจทย์ปัญหาทางธุรกิจอย่างไร
### สรุปความสำคัญ
- **ระยะเวลาการคุยโทรศัพท์ (duration):** คุยนานขึ้นจะเพิ่มโอกาสในการสมัครสมาชิกมากขั้น โดยเป็นปัจจัยที่มีผลสูงสุด
- **ช่องทางการติดต่อ (contact):** การติดต่อทางโทรศัพท์ปกติจะลดโอกาสในการสมัครสมาชิกเมื่อเทียบกับวิธีการอื่น
- **เดือนที่ติดต่อ:** ลูกค้าที่ได้รับการติดต่อในเดือนมีนาคมและสิงหาคมมีแนวโน้มที่จะสมัครมากขึ้น
- **เศรษฐกิจ:** เมื่ออัตราการจ้างงานต่ำและดัชนีราคาผู้บริโภคสูง จะมีโอกาสที่ลูกค้าจะสมัครสมาชิกมากขึ้น

[visit on website](https://warinthon27.wordpress.com/2025/02/19/data-driven-insights-into-bank-campaign-success-what-drives-subscription-rates/?preview_id=171&preview_nonce=4f570612d8&preview=true&_thumbnail_id=163)

