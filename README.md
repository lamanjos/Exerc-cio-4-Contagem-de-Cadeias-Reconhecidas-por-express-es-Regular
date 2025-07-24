# Exerc-cio-4-Contagem-de-Cadeias-Reconhecidas-por-express-es-Regular
import random

def resolver_labirinto(n, m, k, maze, tunnels):
    probabilidade = [[0.0 for _ in range(m)] for _ in range(n)]
    for i in range(n):
        for j in range(m):
            if maze[i][j] == 'A':
                probabilidade[i][j] = 1.0 
    for i1, j1, i2, j2 in tunnels:
        probabilidade[i1][j1] += 0.5
        probabilidade[i2][j2] += 0.5
    for i in range(n):
        for j in range(m):
            if maze[i][j] == 'O':
                probabilidade[i][j] = calcular_probabilidade(i, j, probabilidade, maze)
    probabilidade_maxima = 0.0
    for i in range(n):
        for j in range(m):
            if maze[i][j] == 'O':
                probabilidade_maxima = max(probabilidade_maxima, probabilidade[i][j])
    return probabilidade_maxima

def calcular_probabilidade(i, j, probabilidade, maze):
    vizinhos = [(i-1, j), (i+1, j), (i, j-1), (i, j+1)]
    soma = 0.0
    for x, y in vizinhos:
        if 0 <= x < len(maze) and 0 <= y < len(maze[0]):
            if maze[x][y] == 'O':
                soma += probabilidade[x][y]
            elif maze[x][y] == '#':
                soma += probabilidade[x][y] * 0.5
            elif maze[x][y] == '*':
                soma += probabilidade[x][y] * 0.25
            elif maze[x][y] == '%':
                soma += probabilidade[x][y] * 0.1  
    if soma > 0:
        return soma / len(vizinhos)
    else:
        return 0.0  # Valor seguro se não houver vizinhos válidos

def gerar_labirinto(n, m):
    elementos = ['O'] * 5 + ['#', '*', '%']
    labirinto = [''.join(random.choice(elementos) for _ in range(m)) for _ in range(n)]
    i, j = random.randint(0, n - 1), random.randint(0, m - 1)
    linha = list(labirinto[i])
    linha[j] = 'A'
    labirinto[i] = ''.join(linha)
    return labirinto

def gerar_tuneis(k, n, m):
    tuneis = set()
    while len(tuneis) < k:
        i1, j1 = random.randint(0, n - 1), random.randint(0, m - 1)
        i2, j2 = random.randint(0, n - 1), random.randint(0, m - 1)
        if (i1 != i2 or j1 != j2):
            tuneis.add((i1, j1, i2, j2))
    return list(tuneis)

def main():
    n = int(input("Digite o número de linhas do labirinto (n): "))
    m = int(input("Digite o número de colunas do labirinto (m): "))
    k = int(input("Digite a quantidade de túneis (k): "))

    maze = gerar_labirinto(n, m)
    tunnels = gerar_tuneis(k, n, m)

    print("\nLabirinto:")
    for linha in maze:
        print(linha)
    
    print("\nTúneis:", tunnels)

    resultado = resolver_labirinto(n, m, k, maze, tunnels)
    print(f"\nProbabilidade de fuga: {resultado:.2f}")

if __name__ == '__main__':
    main()
