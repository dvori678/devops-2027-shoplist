## חלק א — מפת ה-cluster

| רכיב | איפה הוא רץ (הנהלה / כל node) | מה הוא עושה, במשפט אחד | אנלוגיה |
|---|---|---|---|
| API server |הנהלה|הכל עובר דרכו|מזכירות|
| etcd |הנהלה|מסד הנתונים עם המצב העכשוי|מערכת הרישום|
| scheduler |הנהלה|מחליט על איזה node ירוץ כל pod חדש|אחראית שיבוצים|
| controller manager |הנהלה|משווה בין המצב הרצוי למצב הנתון ומסדר את הפער|המפקחת|
| kubelet |כל node|מקבל הוראות מההנהלה, דואג שה-pods ב-node שלו רצים כמו שהוגדר ומדווח על מצבם|המחנכת|
| container runtime |כל node|מושך images ומריץ את הקונטיינרים בפועל|המורה בכיתה|
| kube-proxy |כל node|מנתב תעבורת רשת של Services אל ה-pods הנכונים|השליח|
| coredns |הנהלה|מממש DNS|ספר הטלפונים|

## חלק ב — תרחישים

1. בתחילה ה-API server מקבל את הבקשה, וכל הרכיבים מדברים דרכו.
etcd מתעדכן כפי הצורך.
controller manager בודק כמה pods רצויים וכמה יש בפועל.
scheduler מחלק node לכל pod.
kubelet מקבל הוראות מגבוה ובודק אם הכל כמו שצריך להיות.
container runtime מריץ בפועל.
kubelet מעדכן ל-Running.

2. ה-kubelet בבדיקה הקבועה שלו רואה שהקונטיינר קרס,
ומבקש מה-container runtime להריץ אותו מחדש באותו pod.
אם ה-pod נמחק, ה-controller manager יוצר pod חדש, וה-scheduler וה-kubelet מפעילים אותו כמו בתרחיש הקודם

3. ה-kubelet של node 2 מפסיק לדווח ל-API server
ה-controller manager מזהה את השתיקה ומסמן את ה-node כ-NotReady.
אחרי עוד זמן ההנהלה מפנה את ה-pods שהיו על node 2,
controller manager רואה שחסרים pods ויוצר חדשים.
ה-scheduler משבץ אותם על node 1 ו-node 3 שעובדים, וה-kubelet שם מפעיל אותם דרך ה-container runtime.
pod שנוצר בלי Deployment לא יחזור, כי אין מי שאחראי ליצור אותו מחדש.
ב-Docker Desktop יש node אחד שהוא גם ההנהלה וגם מקום ההרצה,
אז כשהמחשב נכבה הכל נעצר ואין שום רכיב שנשאר כדי לתקן.
כשהמחשב עולה שוב, ה-cluster עולה והמצב נשמר ב-etcd על הדיסק,
ולכן ה-pods נוצרים מחדש לפי המצב הרצוי השמור.

4. ה-policy: Never לא נותן למשוך image מבחוץ.
בכיתה ה-image נבנה על המחשב שלי, אבל ב-EKS ה-nodes בענן ואין עליהן את הimages ולכן ה-pod לא יעלה.
כדי לפתור את זה בונים את ה-image ודוחפים אותו ל-registry, ובקובץ ה-YAML משנים את שדה ה-image לכתובת שלו ב-registry ואת policy ל-always.
גם Service מסוג NodePort לא מתאים, כי הוא פותח פורט על ה-IP של ה-node וה-nodes ב-EKS לא נגישים מהאינטרנט. משנים את ה-type ל-LoadBalancer, והוא יוצר מאחורי הקלעים Elastic Load Balancer של AWS שמקבל תעבורה מהאינטרנט ומפזר אותה ל-pods.

5. pod שנוצר עם kubectl run הוא pod בודד בלי בעלים, ולכן כשהוא נמחק אין מי ששם לב לכך ומחזיר אותו.
pod של web נוצר דרך Deployment ו-ReplicaSet, שמחזיק מספר רצוי של pods.
כשנמחק pod, ה-controller manager (דרך ה-ReplicaSet) רואה שיש פחות מהרצוי ויוצר pod חדש מיד.
כדי ש-stats יחזור תמיד, הייתי יוצרת אותו דרך Deployment עם replicas: 1.
