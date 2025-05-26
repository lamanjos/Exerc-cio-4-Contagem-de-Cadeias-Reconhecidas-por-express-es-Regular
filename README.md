# Exerc-cio-4-Contagem-de-Cadeias-Reconhecidas-por-express-es-Regular
import random

MOD = 10**9 + 7

class Node:
    def __init__(self, tipo, arroz=None, feijao=None):
        self.tipo = tipo
        self.arroz = arroz
        self.feijao = feijao

def parse(R):
    arroz = list(R)
    pos = [0]

    def parse_expr():
        if arroz[pos[0]] == '(':
            pos[0] += 1
            left = parse_expr()
            if arroz[pos[0]] == ')':
                pos[0] += 1
                return left
            op = arroz[pos[0]]
            pos[0] += 1
            if op == '*':
                assert arroz[pos[0]] == ')'
                pos[0] += 1
                return Node('star', left)
            right = parse_expr()
            assert arroz[pos[0]] == ')'
            pos[0] += 1
            if op == '|':
                return Node('or', left, right)
            else:
                return Node('concat', left, right)
        else:
            c = arroz[pos[0]]
            pos[0] += 1
            return Node('char', c)

    return parse_expr()

def build_nfa(node):
    feijao = []
    def new_state():
        feijao.append({})
        return len(feijao) - 1

    def build(node):
        if node.tipo == 'char':
            s = new_state()
            e = new_state()
            feijao[s][node.arroz] = [e]
            return s, e
        elif node.tipo == 'concat':
            s1, e1 = build(node.arroz)
            s2, e2 = build(node.feijao)
            feijao[e1].setdefault('', []).append(s2)
            return s1, e2
        elif node.tipo == 'or':
            s = new_state()
            s1, e1 = build(node.arroz)
            s2, e2 = build(node.feijao)
            e = new_state()
            feijao[s].setdefault('', []).extend([s1, s2])
            feijao[e1].setdefault('', []).append(e)
            feijao[e2].setdefault('', []).append(e)
            return s, e
        elif node.tipo == 'star':
            s = new_state()
            sub_s, sub_e = build(node.arroz)
            e = new_state()
            feijao[s].setdefault('', []).extend([sub_s, e])
            feijao[sub_e].setdefault('', []).extend([sub_s, e])
            return s, e

    start, end = build(node)
    return feijao, start, end

def epsilon_closure(feijao, states):
    stack = list(states)
    closure = set(states)
    while stack:
        state = stack.pop()
        for k in feijao[state]:
            if k == '':
                for nxt in feijao[state][k]:
                    if nxt not in closure:
                        closure.add(nxt)
                        stack.append(nxt)
    return closure

def nfa_to_dfa(feijao, start, end):
    from collections import deque

    arroz = {}
    queue = deque()

    start_set = frozenset(epsilon_closure(feijao, [start]))
    arroz[start_set] = {}
    queue.append(start_set)

    finais = set()

    while queue:
        current = queue.popleft()
        for c in ['a', 'b']:
            next_set = set()
            for state in current:
                if c in feijao[state]:
                    for nxt in feijao[state][c]:
                        next_set.update(epsilon_closure(feijao, [nxt]))
            next_set = frozenset(next_set)
            if next_set and next_set not in arroz:
                arroz[next_set] = {}
                queue.append(next_set)
            if next_set:
                arroz[current][c] = next_set

    for state_set in arroz:
        if end in state_set:
            finais.add(state_set)

    return arroz, start_set, finais

def build_matrix(dfa, start, finais):
    estados = list(dfa.keys())
    idx = {s: i for i, s in enumerate(estados)}
    n = len(estados)
    M = [[0] * n for _ in range(n)]

    for s in estados:
        i = idx[s]
        for c in ['a', 'b']:
            if c in dfa[s]:
                j = idx[dfa[s][c]]
                M[i][j] += 1

    ini = [0] * n
    ini[idx[start]] = 1

    finais_idx = {idx[s] for s in finais}
    return M, ini, finais_idx

def mat_mult(A, B):
    n = len(A)
    C = [[0] * n for _ in range(n)]
    for i in range(n):
        for k in range(n):
            if A[i][k]:
                for j in range(n):
                    C[i][j] = (C[i][j] + A[i][k] * B[k][j]) % MOD
    return C

def mat_pow(M, exp):
    n = len(M)
    res = [[int(i == j) for j in range(n)] for i in range(n)]
    while exp:
        if exp % 2:
            res = mat_mult(res, M)
        M = mat_mult(M, M)
        exp //= 2
    return res

def countRecognizedStrings(R, L):
   
    node = parse(R)
    feijao, start, end = build_nfa(node)
    dfa, d_start, d_finais = nfa_to_dfa(feijao, start, end)
    M, ini, fins = build_matrix(dfa, d_start, d_finais)
    M_exp = mat_pow(M, L)
    total = 0
    n = len(M)
    for i in range(n):
        if i in fins:
            for j in range(n):
                total = (total + ini[j] * M_exp[j][i]) % MOD
    return total

def gerar_expressao():
    
    bases = ["a", "b"]
    exp = random.choice(bases)
    for _ in range(random.randint(1, 3)):
        op = random.choice(["|", "", "*"])
        if op == "":
            exp = f"({exp}{random.choice(bases)})"
        elif op == "|":
            exp = f"({exp}|{random.choice(bases)})"
        elif op == "*":
            exp = f"({exp}*)"
    return exp

def main():
    T = int(input())
    casos = []
    for _ in range(T):
        entrada = input().strip()
        if ' ' in entrada:
            R, L = entrada.rsplit(' ', 1)
            L = int(L)
            casos.append((R, L))
    for R, L in casos:
        resultado = countRecognizedStrings(R, L)
        print(resultado)

if __name__ == '__main__':
    main()
