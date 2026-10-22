1. docker build -t greeter:v1 ./lessons/02-docker-basics/homework/greeter
2. docker run -d --name greeter -p 9000:8000 greeter:v1
3. -e GREETING="Shalom"
4. {"message": "Shalom, Rivka!"}
5. רק השורה COPY app.py . תרוץ מחדש, כי רק זה עלול להכיל שינויים