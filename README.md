import random

MOD = 10**9 + 7

def countRecognizedStrings(R, L):
    def parse_expr(s, i=0):
        def parse_atom(s, i):
            # Ignora espaços (se existirem)
            while i < len(s) and s[i] == ' ':
                i += 1
            if i >= len(s):
                raise Exception("Fim inesperado da expressão")

            if s[i] == '(':
                expr, j = parse_expr(s, i + 1)
                while j < len(s) and s[j] == ' ':
                    j += 1
                if j >= len(s) or s[j] != ')':
                    raise Exception("Parênteses não balanceados")
                j += 1
                while j < len(s) and s[j] == ' ':
                    j += 1
                if j < len(s) and s[j] == '*':
                    return ('star', expr), j + 1
                return expr, j
            elif s[i] in ('a', 'b'):
                j = i + 1
                while j < len(s) and s[j] == ' ':
                    j += 1
                if j < len(s) and s[j] == '*':
                    return ('star', ('char', s[i])), j + 1
                return ('char', s[i]), j
            else:
                raise Exception(f"Caractere inválido no parser: '{s[i]}'")

        nodes = []
        j = i
        while j < len(s):
            if s[j] == ')':
                break
            if s[j] == '|':
                break
            node, j = parse_atom(s, j)
            nodes.append(node)
            while j < len(s) and s[j] == ' ':
                j += 1

        if not nodes:
            raise Exception("Expressão vazia")

        left = nodes[0]
        for node in nodes[1:]:
            left = ('concat', left, node)

        if j < len(s) and s[j] == '|':
            right, j2 = parse_expr(s, j + 1)
            return ('alt', left, right), j2

        return left, j

    tree, pos = parse_expr(R, 0)
    if pos != len(R):
        raise Exception("Parser não consumiu toda a expressão")

    memo = {}
    def count(node, length):
        if (node, length) in memo:
            return memo[(node, length)]

        t = node[0]
        if t == 'char':
            res = 1 if length == 1 else 0
        elif t == 'concat':
            left, right = node[1], node[2]
            res = 0
            for i in range(length + 1):
                res += count(left, i) * count(right, length - i)
            res %= MOD
        elif t == 'alt':
            left, right = node[1], node[2]
            res = (count(left, length) + count(right, length)) % MOD
        elif t == 'star':
            child = node[1]
            dp = [0] * (length + 1)
            dp[0] = 1
            for l in range(1, length + 1):
                s = 0
                for k in range(1, l + 1):
                    s += count(child, k) * dp[l - k]
                dp[l] = s % MOD
            res = dp[length]
        else:
            res = 0

        memo[(node, length)] = res
        return res

    return count(tree, L)

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
    T = int(input("Digite o número de casos de teste: "))
    k = int(input("Digite o valor máximo de L: "))
    casos = []
    for _ in range(T):
        R = gerar_expressao()
        L = random.randint(1, k)
        casos.append((R, L))
    
    for R, L in casos:
        print(f"Expressão: {R}, Tamanho: {L}")
        resultado = countRecognizedStrings(R, L)
        print("Reconhecidas:", resultado)

if __name__ == '__main__':
    main()
