# ד10 — תשובות

### 1.
נתנו ל־Images שמות קבועים משלנו כדי שה־Deployments יוכלו להשתמש בהם בלי להיות תלויים בשמות ש־Docker Compose יצר. ללא `imagePullPolicy: Never`, Kubernetes עלול לנסות למשוך את ה־Image מ־Registry במקום להשתמש ב־Image המקומי.

### 2.
פענחנו את הסיסמה באמצעות הפקודה:
```powershell
[Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("c2hvcHBhc3M="))
```
התוצאה היא `shoppass`. למדנו ש־Base64 אינו הצפנה, ולכן Secret אינו מאובטח רק בזכות הקידוד שלו.

### 3.
הקובץ `postgres-pvc.yml` אחראי לבקשת האחסון המתמיד, ששומר על הנתונים גם לאחר מחיקת ה־Pod. הפקודה שהייתה מוחקת את בקשת האחסון היא:
```powershell
kubectl delete pvc postgres-pvc -n shoplist
```

### 4.
אם ה־Backend עולה לפני PostgreSQL, הוא עשוי להיכשל בחיבור למסד הנתונים. Kubernetes משתמש ב־Readiness Probe כדי למנוע העברת בקשות ל־Backend כל עוד בדיקת המוכנות נכשלת.

### 5.
שינינו את ה־Selector של ה־Service מ־`app: backend` ל־`app: backnd`. ה־Pod נשאר במצב Running, אבל ה־Endpoints הפכו ל־`<none>`, כי ה־Service לא מצא Pod עם התווית המתאימה.

### 6.
שינוי ConfigMap אינו מעדכן משתני סביבה שכבר נטענו בתוך Pods קיימים. הפתרון הוא להפעיל מחדש את ה־Deployment באמצעות:
```powershell
kubectl rollout restart deployment/backend -n shoplist
```

### 7.
מצאנו 17 הופעות של `shoplist` ב־12 קובצי המטלה. כדי לעבור ל־namespace בשם `shoplist-test`, נצטרך לשנות את הגדרות ה־namespace בקבצים הרלוונטיים.