
#בחירת image ל python
FROM python:3.11-slim

#הגדרת תיקית העבודה
WORKDIR /lessons/02-docker-basics/homework/greeter

#העתקת קובץ הדרישות
COPY requirements.txt .

#התקנת החבילות
RUN pip install --no-cache-dir -r requirements.txt

#העתקת קובץ הקוד
COPY app.py .

#הצהרה על הפורט
EXPOSE 8000

#הפעלה באמצעות שורת הפקודה
CMD ["python", "app.py"]