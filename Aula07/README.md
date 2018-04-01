## Matemática

### Divisores

Um problema recorrente é o de encontrar divisores de um número positivo. A maneira mais simples de resolvê-lo seria passar por todos os números e testar se o resto da divisão é 0, ou seja, se é divisível

```c++
vector<long long> all_divisors(long long n){
  vector<long long> ans;
  for(long long i = 1; i <= n; i++)
    if(n % i == 0)
      ans.push_back(i);
  return ans;
}```

é fácil ver que a complexidade do código acima é `O(n)`, podemos fazer melhor que isso com algumas observações.

Se `a` é um divisor `n` então o resto da divisão de `n` por `a` é 0 assim `b = n/a` é um inteiro. Sabemos então que `a*b = n`, ou seja, `a = n/b` e assim `b` também é um divisor de `n`. Se fixarmos que `a <= b`, qual o valor máximo de `a`? Como `a` é no máximo `b`, consideremos o caso em que `a = b` temos que `a*a = n`, ou seja, `a = sqrt(n)`.

Agora é possivel modificar o código passando por todos os valores possíveis de `a` e computar o respectivo `b` para encontrar todos os divisores.

```c++
vector<long long> all_divisors(long long n){
  vector<long long> ans;
  for(long long a = 1; a*a <= n; a++){ // comparação que evita o uso de doubles, a <= sqrt(n) é o mesmo que a*a <= n, ja que a e n sao positivos
    if(n % a == 0){
      long long b = n / a;
      ans.push_back(a);
      ans.push_back(b);
    }
  }
  sort(ans.begin(), ans.end()); // frescura para retornar os divisores ordenados como na primeira implementação
  return ans;
}```

Só há um problema com a implementação acima. Assumimos que `a <= b`, caso `a = b` inserimos o divisor 2 vezes na resposta, por exemplo, para 36 podemos ter `a = 6` e `b = 6`. Assim a versão final do código fica:

```c++
vector<long long> all_divisors(long long n){
  vector<long long> ans;
  for(long long a = 1; a*a <= n; a++){ // comparação que evita o uso de doubles, a <= sqrt(n) é o mesmo que a*a <= n
    if(n % a == 0){
      long long b = n / a;
      ans.push_back(a);
      if(a != b) ans.push_back(b);
    }
  }
  sort(ans.begin(), ans.end()); // frescura para retornar os divisores ordenados como na primeira implementação
  return ans;
}```

com complexidade `O(sqrt(n))`.

###### Observações
Um número primo tem somente dois divisores positivos, assim podemos checar se um numero `x` é primo usando `all_divisors(x).size() == 2` ou modificando um pouco a rotina e ter uma melhor constante na complexidade
```c++
vector<long long> is_prime(long long n){
  if(n == 1) return 0;
  for(long long a = 2; a*a <= n; a++){ // comparação que evita o uso de doubles, a <= sqrt(n) é o mesmo que a*a <= n
    if(n % a == 0){
      return 0;
    }
  }
  return 1;
}```

## Todos os múltiplos de todos os números até N

### Todos os múltiplos de x até N
Consideramos multiplos de `x` os números: `x, 2*x, 3*x, 4*x, ...` ou, escrevendo de outra forma, `x, x+x, x+x+x, x+x+x+x, ...`

Caso queiramos fazer algo com todos os múltiplos de `x` até um limite `N` podemos usar a simples rotina
```c++
for(int m = x; m < N; m += x){ // m é sempre multiplo de x
  // code
}
```
Que é executada em `O(N/x)`.

### Todos os múltiplos de todos os números até N

Se passarmos por todos os números `x` entre 1 e `N` e para cada um deles achar todos os múltiplos `m`.

O código ficaria algo como

```c++
for(int x = 1; x < N; x++){
  for(int m = x; m < N; m += x){ // m é sempre multiplo de x
    // code
  }
}```

O código acima parece ser executado em `O(N^2)`, mas podemos definir uma cota bem menor, com algumas observações. O código é executado em `N/1 + N/2 + ... + N/(N-1) + N/N` passos. Podemos botar o `N` em evidencia `N*(1/1 + 1/2 + 1/3 + ... + 1/(N-1) + 1/N)`. A soma dentro dos parenteses é menor que a área abaixo da curva da função `1/x`, a integral é `ln(x)`(mas relaxa que não precisa lembrar das coisas de cálculo 1). Portanto `O(N*(1/1 + 1/2 + 1/3 + ... + 1/(N-1) + 1/N)) = O(N*lg N)`.

Podemos resolver vários problemas usando isso pois `x` será divisor de `m` e assim para todo `m` também passaremos por todos os divisores deles. Por exemplo, para encontrar todos os primos ate `N` podemos contar a quantidade de divisores que cada elemento tem e no final ver quais tem quantidade 2.
```c++
vector<int> primos_ate_n(int N){
  vector<int> qnt_div(N, 0);
  for(int x = 1; x < N; x++){
    for(int m = i; m < N; m += x){
      qnt_div[m]++; // aqui descobrimos que x é divisor de m
    }
  }
  vector<int> primos;
  for(int x = 1; x < N; x++){
    if(qnt_div[x] == 2)
      primos.push_back(x);
  }
  return primos;
}
```

Ou usar um outra ideia, marcar inicialmente todos os números entre 1 e `N` como possiveis primos. Passando em ordem crescente e quando encontramos um primo marcamos os múltiplos do primo como não primos.

```c++
vector<int> primos_ate_n(int N){
  vector<int> marcacao(N, 1); // 1 = possivel primo, 0 = com certeza não primo
  vector<int> primos;
  for(int x = 2; x < N; x++) if(marcacao[x] == 1){
    primos.push_back(x);
    for(int m = i+i; m < N; m += x){
      marcacao[m] = 0; // aqui descobrimos que m não é primo
    }
  }
  return primos;
}
```


## Fatoração

A primeira forma seria algo como

```c++
// retorna vetor de pair<primo, expoente> da fatoração
// fatora(36) = [{2, 2}, {3, 2}]
vector<pair<long long, int>> fatora(long long n){
  vector<pair<long long, int>> ans;
  for(long long p = 2; p <= n; p++){
    if(n % p == 0){
      int expoente = 0;
      while(n % p == 0){
        n /= p;
        expoente++;
      }
      ans.emplace_back(a, expoente);
    }
  }
  return ans;
}```

A primeira vista parece que temos que testar se `p` é primo. Entretanto passamos por `p` de forma crescente e sempre que podemos dividimos `n` por `p` então a condição `(n % p == 0)` só será verdade para `p` primos.

Apesar do código acima rodar bem para vários exemplos, no pior caso `n` é primo e o código é executado em `O(n)`.

Podemos melhorar a complexidade com uma simples observação. É possivel ter um primo maior que a `sqrt(n)`, por exemplo, 10 tem 5 como fator e `5 > sqrt(10)`, mas é impossível ter dois primos maiores que a raiz. Se tivermos `a > sqrt(n)` e `b > sqrt(n)`, quando multiplicamos temos que `a * b > sqrt(n) * sqrt(n)` e `a * b > n`.

```c++
vector<pair<long long, int>> fatora(long long n){
  vector<pair<long long, int>> ans;
  for(long long p = 2; p*p <= n; p++){ // comparação que evita o uso de doubles, p <= sqrt(n) é o mesmo que p*p <= n
    if(n % p == 0){
      int expoente = 0;
      while(n % p == 0){
        n /= p;
        expoente++;
      }
      ans.emplace_back(a, expoente);
    }
  }
  if(n > 1) ans.emplace_back(n, 1);
  return ans;
}```

### Fatoração em O(lg n) para números até N

É possível fatores números ate um limite `N` em `O(lg n)` após preprocessamento `O(n lg n)`.

Para isto computamos o vetor `lp` de tamanho `N` onde `lp[x] = algum primo divisor de x`. Podemos fazê-lo passando por todos os multiplos de primos.

```c++
vector<int> lp(N, -1);

for(int x = 2; x < N; x++)
  if(lp[x] == -1){ // se x nao foi marcado antes, é primo
    for(int m = x; m < N; m += x) // todos os multiplos de i
      lp[m] = x;
  }

```

Tendo este vetor podemos fatorar um numero `x` com o seguinte procedimento

```c++
vector<pair<int, int>> fatora(int x){
  map<int, int> expoentes;
  while(x > 1){
    expoente[ lp[x] ]++; // aumentamos o expoente do primo lp[x] em 1 na resposta
    x /= lp[x];
  }
  vector<pair<int, int>> ans;
  for(pair<int, int> p : expoentes)
    ans.emplace_back(p);
  return ans;
}
```

A complexidade do procedimento acima é `O(quantidade de fatores)`, que é limitado por `O(lg n)`, da para ver que no pior caso todos os fatores são 2(menor primo) e a complexidade é o `k` de `2^k = n`.

## Fatoração de n!

Para encontrar o expoente de um primo `p` na fatoração de `n!`. Contamos a quantidade de vezes que aparece um número ate `n` com expoente pelo menos 1, depois a quantidade de vezes que aparece um número ate `n` com expoente pelo menos 2 e assim por diante. Ou seja, `n/p + n/(p^2) + n/(p^3) + ...` considerando a divisão inteira. A rotina em c++ seria:

```c++
long long expoente_em_fatorial(long long p, long long n){
  long long ans = 0, tmp = p;

  while(tmp <= n){ // enquanto floor(n/tmp) não da 0
    ans += n / tmp;
    tmp *= p; // muda de p para p^2, de p^2 para p^3 e assim por diante
  }

  return ans;
}
```
