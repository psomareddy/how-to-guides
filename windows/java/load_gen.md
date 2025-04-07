## Locust Load Generator for PetClinic app

Create a empty text file named locustfile.py
```
notepad locustfile.py
```

and copy the following contents into that file
```locustfile.py
import time
from locust import HttpUser, task, between

class QuickstartUser(HttpUser):
    wait_time = between(1, 5)

    def on_start(self):
        self.client.get("/")

    @task(3)
    def view_items(self):
        for item_id in range(1, 10):
            self.client.get(f"/owners/{item_id}", name="/owners")
            time.sleep(1)
```

Run a docker container that executes this load script

```locust.cmd
docker run --network="host" -d -p 8089:8089 -v .:/mnt/locust docker.io/locustio/locust -f /mnt/locust/locustfile.py --headless -u 10 -r 3 -H http://127.0.0.1:8080
```