import pandas as pd
import matplotlib.pyplot as plt
import time
import sympy
import random

# Criando dados para salvar no CSV com 10 testes por chave
results = []
key_sizes = [1024, 2048, 4096]

def generate_large_prime(bits):
    return sympy.randprime(2**(bits-1), 2**bits)

def mod_inverse(a, m):
    return pow(a, -1, m)

def rsa_decrypt_direct(c, d, n):
    start_time = time.time()
    pow(c, d, n)
    return time.time() - start_time

def rsa_decrypt_trc(c, d, p, q):
    start_time = time.time()
    mp = pow(c, d, p)
    mq = pow(c, d, q)
    q_inv = mod_inverse(q, p)
    p_inv = mod_inverse(p, q)
    (mp * q * q_inv + mq * p * p_inv) % (p * q)
    return time.time() - start_time

for bits in key_sizes:
    for _ in range(100):  # Executar 100 testes por tamanho de chave
        p = generate_large_prime(bits // 2)
        q = generate_large_prime(bits // 2)
        n = p * q
        phi_n = (p - 1) * (q - 1)
        e = 65537
        d = mod_inverse(e, phi_n)
        c = random.randint(2, n-1)
        time_direct = rsa_decrypt_direct(c, d, n)
        time_trc = rsa_decrypt_trc(c, d, p, q)
        results.append([bits, time_direct, time_trc])

# Criar DataFrame e salvar os resultados em um arquivo CSV
df_results = pd.DataFrame(results, columns=["Tamanho da Chave (bits)", "Tempo Direto (s)", "Tempo TRC (s)"])
df_results.to_csv("resultados_rsa_trc.csv", index=False)
print("Arquivo CSV salvo como 'resultados_rsa_trc.csv'")

# --- Gerando o gráfico a partir do CSV ---

def plot_graph_from_csv(csv_filename):
    df = pd.read_csv(csv_filename)
    grouped = df.groupby("Tamanho da Chave (bits)").mean()
    key_sizes = grouped.index.tolist()
    times_direct = grouped["Tempo Direto (s)"].tolist()
    times_trc = grouped["Tempo TRC (s)"].tolist()
    plt.figure(figsize=(10, 6))
    plt.plot(key_sizes, times_direct, marker='o', linestyle='-', label="Decodificação Direta")
    plt.plot(key_sizes, times_trc, marker='s', linestyle='-', label="Decodificação com TRC")
    plt.xlabel("Tamanho da Chave (bits)")
    plt.ylabel("Tempo de Execução (segundos)")
    plt.title("Comparação de Desempenho: Decodificação RSA Direta vs. TRC (Média de 100 Testes)")
    plt.legend()
    plt.grid(True)
    plt.show()

plot_graph_from_csv("resultados_rsa_trc.csv")
