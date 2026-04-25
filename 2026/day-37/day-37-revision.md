## Quick Answers

1. Image vs Container:
Image = blueprint, Container = running instance

2. Data after container delete:
Lost unless volume is used

3. Communication:
Via container name (DNS) in custom network

4. docker compose down -v:
Removes containers + volumes

5. Multi-stage:
Reduces image size by copying only required files

6. COPY vs ADD:
COPY = simple copy, ADD = extra features (extract, URL)

7. -p 8080:80:
Host 8080 → Container 80

8. Disk usage:
docker system df
