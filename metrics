import threading
import time
import random

class Metrics:
    def __init__(self):
        self.total_throughput = 0
        self.lock = threading.Lock()
        self.cpu_usage = {
            "RU": 0,
            "DU": 0,
            "CU": 0,
            "RIC": 0
        }
        self.jitter_values = []

    def update_throughput(self, throughput):
        with self.lock:
            self.total_throughput += throughput

    def update_cpu_usage(self, component, usage):
        with self.lock:
            # Simple running average calculation
            self.cpu_usage[component] = (self.cpu_usage[component] + usage) / 2

    def add_jitter(self, jitter):
        with self.lock:
            self.jitter_values.append(jitter)

    def report(self):
        avg_jitter = sum(self.jitter_values) / len(self.jitter_values) if self.jitter_values else 0
        print("\n=== Metrics Report ===")
        print(f"Total Throughput: {self.total_throughput:.2f} Mbps")
        print(f"CPU Usage per Component:")
        for comp, usage in self.cpu_usage.items():
            print(f"  {comp}: {usage:.2f}%")
        print(f"Average Jitter: {avg_jitter*1000:.2f} ms")
        print("======================\n")

class RadioUnit:
    def __init__(self, ru_id, metrics):
        self.ru_id = ru_id
        self.metrics = metrics

    def send_signal(self, data):
        if random.random() < 0.05:  # 5% packet loss
            print(f"[RU-{self.ru_id}] Packet lost while sending: {data}")
            return None
        latency = random.uniform(0.01, 0.1)
        time.sleep(latency)
        self.metrics.add_jitter(latency)
        cpu_load = random.uniform(5, 15)  # RU CPU usage %
        self.metrics.update_cpu_usage("RU", cpu_load)
        throughput = random.uniform(1, 100)  # Mbps
        self.metrics.update_throughput(throughput)
        print(f"[RU-{self.ru_id}] Sending signal: {data} | Throughput: {throughput:.2f} Mbps | CPU load: {cpu_load:.2f}%")
        return data

    def receive_signal(self, data):
        if data:
            latency = random.uniform(0.01, 0.05)
            time.sleep(latency)
            self.metrics.add_jitter(latency)
            print(f"[RU-{self.ru_id}] Received signal: {data}")

class DistributedUnit:
    def __init__(self, du_id, metrics):
        self.du_id = du_id
        self.metrics = metrics

    def process_signal(self, data):
        latency = random.uniform(0.02, 0.08)
        time.sleep(latency)
        self.metrics.add_jitter(latency)
        cpu_load = random.uniform(10, 30)  # DU CPU usage %
        self.metrics.update_cpu_usage("DU", cpu_load)
        print(f"[DU-{self.du_id}] Processing signal: {data} -> Processed({data}) | CPU load: {cpu_load:.2f}%")
        return f"Processed({data})"

class CentralUnit:
    def __init__(self, cu_id, metrics):
        self.cu_id = cu_id
        self.metrics = metrics

    def manage_connection(self, user_data):
        latency = random.uniform(0.01, 0.03)
        time.sleep(latency)
        self.metrics.add_jitter(latency)
        cpu_load = random.uniform(8, 20)  # CU CPU usage %
        self.metrics.update_cpu_usage("CU", cpu_load)
        print(f"[CU-{self.cu_id}] Managing user session for: {user_data} | CPU load: {cpu_load:.2f}%")

class RIC:
    def __init__(self, metrics):
        self.policy = "Load Balancing"
        self.metrics = metrics

    def apply_policy(self, component, data):
        cpu_load = random.uniform(2, 10)  # RIC CPU usage %
        self.metrics.update_cpu_usage("RIC", cpu_load)
        print(f"[RIC] Applying {self.policy} policy on {component}: {data} | CPU load: {cpu_load:.2f}%")

def simulate_user_session(user_id, ru, du, cu, ric):
    user_data = f"UE{user_id}-data"
    ric.apply_policy("RU", user_data)

    signal = ru.send_signal(user_data)
    if signal is None:
        print(f"[UE{user_id}] Session aborted due to packet loss.")
        return

    processed = du.process_signal(signal)
    cu.manage_connection(processed)
    ru.receive_signal(f"ACK for {user_data}")
    print("-" * 50)

def simulate_o_ran_concurrent_users(user_count=100):
    metrics = Metrics()
    ru = RadioUnit(ru_id=1, metrics=metrics)
    du = DistributedUnit(du_id=101, metrics=metrics)
    cu = CentralUnit(cu_id=201, metrics=metrics)
    ric = RIC(metrics=metrics)

    threads = []

    for user_id in range(1, user_count + 1):
        t = threading.Thread(target=simulate_user_session, args=(user_id, ru, du, cu, ric))
        threads.append(t)
        t.start()

    for t in threads:
        t.join()

    metrics.report()

if __name__ == "__main__":
    simulate_o_ran_concurrent_users(100)
