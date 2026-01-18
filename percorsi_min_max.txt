import networkx as nx

# =====================================================
# 1) CREAZIONE DI UN GRAFO DIRETTO PESATO (DAG)
# =====================================================

# Grafo di esempio:
# A → B (3)
# A → C (2)
# B → D (4)
# C → D (1)

G = nx.DiGraph()
G.add_weighted_edges_from([
    ('A', 'B', 3),
    ('A', 'C', 2),
    ('B', 'D', 4),
    ('C', 'D', 1),
])

# =====================================================
# 2) VERIFICA CHE SIA UN DAG (necessario per il percorso massimo)
# =====================================================

print(nx.is_directed_acyclic_graph(G))  # True

# =====================================================
# 3) PERCORSO MINIMO
# =====================================================

# --- 3a) Percorso minimo in termini di PESO ---
# NetworkX usa Dijkstra (grafi pesati senza pesi negativi)

min_path = nx.shortest_path(
    G,
    source='A',
    target='D',
    weight='weight'
)

min_cost = nx.shortest_path_length(
    G,
    source='A',
    target='D',
    weight='weight'
)

print("Percorso minimo:", min_path)
print("Costo minimo:", min_cost)
# Risultato:
# Percorso: A → C → D
# Costo: 2 + 1 = 3

# Oppure
def getPercorsoMinimo(self, idStazPartenza, idStazArrivo):
    vSource = self._dizionario_fermate[idStazPartenza]
    vTarget = self._dizionario_fermate[idStazArrivo]

    costo, percorso = nx.single_source_dijkstra(self._grafo, vSource, vTarget, weight='tempo')
    return costo, percorso


# =====================================================
# Nodi raggiungibili a partire da un nodo di partenza
# =====================================================

def getRaggiungibili(self, idStazPartenza):
    fermataPartenza = self._dizionario_fermate[idStazPartenza]

    edges = nx.bfs_edges(self._grafo, fermataPartenza)  # Archi del grafo di visita da fermataPartenza
    visited_nodes = []
    for u, v in edges:
        visited_nodes.append(v)
    return visited_nodes


# =====================================================
# Componente connessa
# =====================================================

def get_component(self, album):
     """Restituisce la componente connessa di un album"""
     componente = list(nx.node_connected_component(self.G, int(album)))
     #componente.remove(int(album))
     return componente

# =====================================================
# Archi entranti e uscenti in un DiGraph
# =====================================================
def calcola_peso_nodo(self, product_id):
    valore = 0
    for arco_out in self.G.out_edges(product_id, data=True):
            valore = valore + arco_out[2]["weight"]
    for arco_in in self.G.in_edges(product_id, data=True):
            valore = valore - arco_in[2]["weight"]
    return valore

# oppure

#Grafo ORIENTATO (DiGraph):
#- successors(n)   → nodi raggiungibili da n
#- predecessors(n) → nodi che arrivano in n
#- neighbors(n)    → come successors (ma sconsigliato)

#Grafo NON orientato (Graph):
#- neighbors(n)    → nodi adiacenti


# =====================================================
# 4) PERCORSO MASSIMO (solo perché il grafo è un DAG!)
# =====================================================

# --- 4a) Percorso massimo globale nel DAG ---
max_path = nx.dag_longest_path(G, weight='weight')
max_cost = nx.dag_longest_path_length(G, weight='weight')

print("Percorso massimo:", max_path)
print("Costo massimo:", max_cost)
# Risultato:
# Percorso: A → B → D
# Costo: 3 + 4 = 7


# =====================================================
# 5) PERCORSO MASSIMO TRA DUE NODI (A → D)
# =====================================================

# 1) Nodi raggiungibili da A
from_A = nx.descendants(G, 'A') | {'A'}

# 2) Nodi che possono arrivare a D
to_D = nx.ancestors(G, 'D') | {'D'}

# 3) Sottografo che contiene solo cammini A → D
H = G.subgraph(from_A & to_D)

# 4) Longest path sul sottografo
max_path_AD = nx.dag_longest_path(H, weight='weight')
max_cost_AD = nx.dag_longest_path_length(H, weight='weight')

print("Percorso massimo A→D:", max_path_AD)
print("Costo massimo A→D:", max_cost_AD)


# =====================================================
# 6) COSA SUCCEDE SE USO DFS (solo a scopo didattico)
# =====================================================

# DFS NON garantisce né percorso minimo né massimo
dfs_edges = list(nx.dfs_edges(G, source='A'))
print("DFS edges:", dfs_edges)


# =====================================================
# 7) RICORSIONE (schema "universale")
# =====================================================
def trova_cammino_ottimo(self, start, end, L):
    """
    Funzione esterna:
    - inizializza la soluzione migliore
    - avvia la ricorsione
    """
    self.best_sol = None
    self.best_value = float("-inf")

    soluzione_parziale = [start]
    visitati = {start}
    valore_corrente = 0

    self._ricorsione(
        nodo_corrente=start,
        nodo_finale=end,
        L=L,
        soluzione_parziale=soluzione_parziale,
        visitati=visitati,
        valore_corrente=valore_corrente
    )

    return self.best_sol, self.best_value


def _ricorsione(self, nodo_corrente, nodo_finale, L,
                soluzione_parziale, visitati, valore_corrente):
    """
    Funzione ricorsiva universale
    """

    # 🔹 CASO BASE
    if self._condizione_terminazione(soluzione_parziale, L):
        if self._soluzione_valida(nodo_corrente, nodo_finale):
            if self._migliora_soluzione(valore_corrente):
                self.best_sol = soluzione_parziale.copy()
                self.best_value = valore_corrente
        return

    # 🔹 GENERAZIONE DELLE SCELTE
    for scelta in self._genera_scelte(nodo_corrente):

        if self._vincoli(scelta, visitati):

            # SCELTA
            soluzione_parziale.append(scelta)
            visitati.add(scelta)
            incremento = self._valore_scelta(nodo_corrente, scelta)

            self._ricorsione(
                nodo_corrente=scelta,
                nodo_finale=nodo_finale,
                L=L,
                soluzione_parziale=soluzione_parziale,
                visitati=visitati,
                valore_corrente=valore_corrente + incremento
            )

            # BACKTRACKING
            soluzione_parziale.pop()
            visitati.remove(scelta)
