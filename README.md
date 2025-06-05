import random
import heapq
import itertools
import matplotlib.pyplot as plt
import networkx as nx

# ----- Graph Implementation -----
class Graph:
    def __init__(self):
        self.graph = {}

    def add_vertex(self, name):
        if name not in self.graph:
            self.graph[name] = {}

    def add_edge(self, from_vertex, to_vertex, weight):
        self.graph[from_vertex][to_vertex] = weight
        self.graph[to_vertex][from_vertex] = weight

    def dijkstra(self, start, end):
        queue = [(0, start, [])]
        visited = set()
        while queue:
            (cost, current, path) = heapq.heappop(queue)
            if current in visited:
                continue
            path = path + [current]
            visited.add(current)
            if current == end:
                return cost, path
            for neighbor, weight in self.graph[current].items():
                if neighbor not in visited:
                    heapq.heappush(queue, (cost + weight, neighbor, path))
        return float("inf"), []

    def tsp_brute_force(self, start=None):
        cities = list(self.graph.keys())
        if start:
            cities.remove(start)
        else:
            start = cities[0]
            cities = cities[1:]

        min_path = None
        min_cost = float("inf")
        for perm in itertools.permutations(cities):
            cost = 0
            path = [start] + list(perm)
            valid = True
            for i in range(len(path) - 1):
                if path[i + 1] in self.graph[path[i]]:
                    cost += self.graph[path[i]][path[i + 1]]
                else:
                    valid = False
                    break
            if valid and start in self.graph[path[-1]]:
                cost += self.graph[path[-1]][start]
                if cost < min_cost:
                    min_cost = cost
                    min_path = path + [start]
        return min_cost, min_path if min_path else []

    def hitung_jarak_rute(self, rute):
        total = 0
        for i in range(len(rute) - 1):
            asal, tujuan = rute[i], rute[i + 1]
            if tujuan in self.graph[asal]:
                total += self.graph[asal][tujuan]
            else:
                print(f"Tidak ada jalan langsung antara {asal} dan {tujuan}.")
                return float("inf")
        return total

# ----- Fungsi URL Google Maps -----
def buat_url_google_maps(rute):
    base_url = "https://www.google.com/maps/dir/"
    return base_url + "/".join(rute).replace(' ', '+')

# ----- Generate the Graph -----
cities = [
    'Surabaya', 'Malang', 'Kediri', 'Blitar', 'Madiun',
    'Jember', 'Banyuwangi', 'Lamongan', 'Tuban', 'Probolinggo'
]

g = Graph()
for city in cities:
    g.add_vertex(city)

# Buat graf dasar terhubung (rantai)
for i in range(len(cities) - 1):
    dist = random.randint(50, 200)
    g.add_edge(cities[i], cities[i + 1], dist)

# Tambah edge acak hingga total 30 edge
edges_added = set()
for i in range(len(cities) - 1):
    edges_added.add((cities[i], cities[i + 1]))

while len(edges_added) < 30:
    c1, c2 = random.sample(cities, 2)
    if c1 != c2 and (c1, c2) not in edges_added and (c2, c1) not in edges_added:
        dist = random.randint(50, 300)
        g.add_edge(c1, c2, dist)
        edges_added.add((c1, c2))

# ----- Jalankan Dijkstra -----
start_city = 'Surabaya'
end_city = 'Banyuwangi'
dijkstra_distance, dijkstra_path = g.dijkstra(start_city, end_city)

# ----- Jalankan TSP -----
tsp_distance, tsp_path = g.tsp_brute_force()

# ----- Visualisasi Graf -----
G_nx = nx.Graph()
for from_city in g.graph:
    for to_city, weight in g.graph[from_city].items():
        if (to_city, from_city) not in G_nx.edges:
            G_nx.add_edge(from_city, to_city, weight=weight)

plt.figure(figsize=(12, 8))
pos = nx.spring_layout(G_nx, seed=42)
nx.draw(G_nx, pos, with_labels=True, node_color='lightgreen', node_size=2000, font_size=10)
edge_labels = nx.get_edge_attributes(G_nx, 'weight')
nx.draw_networkx_edge_labels(G_nx, pos, edge_labels=edge_labels)
plt.title("Graf Kota di Jawa Timur (10 Kota, 30 Edge)")
plt.show()

# ----- Output Dijkstra -----
print("\n=== Dijkstra ===")
print(f"Jalur tercepat dari {start_city} ke {end_city}:")
print(" → ".join(dijkstra_path))
print(f"Total jarak: {dijkstra_distance} km")
print("🔗 Lihat rute di Google Maps:")
print(buat_url_google_maps(dijkstra_path))

# ----- Output TSP -----
print("\n=== Traveling Salesman Problem (TSP) ===")
if tsp_path:
    print("Rute optimal (mengunjungi semua kota 1x dan kembali):")
    print(" → ".join(tsp_path))
    print(f"Total jarak: {tsp_distance} km")
    print("🔗 Lihat rute TSP di Google Maps:")
    print(buat_url_google_maps(tsp_path))
else:
    print("Tidak ditemukan rute TSP valid (beberapa kota tidak saling terhubung langsung).")

# ----- Hitung jarak dari input asal dan tujuan -----
print("\n=== Hitung Jarak dari Input Manual (Dijkstra) ===")
print(f"Daftar kota yang tersedia: {', '.join(cities)}")
asal_input = input("Masukkan kota asal: ").strip()
tujuan_input = input("Masukkan kota tujuan: ").strip()

if asal_input not in cities or tujuan_input not in cities:
    print("Kota asal atau tujuan tidak valid.")
elif asal_input == tujuan_input:
    print("Kota asal dan tujuan tidak boleh sama.")
else:
    jarak_manual, path_manual = g.dijkstra(asal_input, tujuan_input)
    if jarak_manual == float("inf"):
        print(f"Tidak ditemukan jalur dari {asal_input} ke {tujuan_input}.")
    else:
        print(f"Jalur tercepat dari {asal_input} ke {tujuan_input}:")
        print(" → ".join(path_manual))
        print(f"Total jarak: {jarak_manual} km")
        print("🔗 Lihat rute di Google Maps:")
        print(buat_url_google_maps(path_manual))
