
# Projet : Série de Fibonacci - Simulation avec ModelSIM et Quartus Prime Lite

## Description

Ce projet implémente un calculateur de la série de Fibonacci à l'aide de VHDL, en utilisant une machine à états (FSM) pour effectuer les calculs. L'architecture comporte trois états :

1. **Idle** : L'état d'attente où le système est prêt à démarrer les calculs.
2. **Op** : L'état de calcul où les nombres de Fibonacci sont calculés.
3. **Done** : L'état final indiquant que le calcul est terminé.

### Objectif
Le but du laboratoire est de concevoir et de simuler un module en VHDL qui calcule le n-ième nombre de Fibonacci, en utilisant un nombre de 5 bits pour l'entrée et un nombre de 20 bits pour la sortie. Le testbench fourni permet de simuler l'algorithme, et le chronogramme vous permet de visualiser l'évolution du calcul.

## Code VHDL

### Entité : `fib`

Le code VHDL de l'entité `fib` est implémenté dans l'architecture suivante :

```vhdl
LIBRARY ieee;
USE ieee.std_logic_1164.ALL;
USE ieee.numeric_std.ALL;

ENTITY fib IS
   PORT (
      clk, reset : IN STD_LOGIC;
      start : IN STD_LOGIC;
      i : IN STD_LOGIC_VECTOR(4 DOWNTO 0);
      ready : OUT STD_LOGIC;
      done_tick : OUT STD_LOGIC;
      f : OUT STD_LOGIC_VECTOR(19 DOWNTO 0)
   );
END fib;

ARCHITECTURE arch OF fib IS
   TYPE state_type IS (idle, op, done);
   SIGNAL state_reg, state_next : state_type;
   SIGNAL t0_reg, t0_next : unsigned(19 DOWNTO 0);
   SIGNAL t1_reg, t1_next : unsigned(19 DOWNTO 0);
   SIGNAL n_reg, n_next : unsigned(4 DOWNTO 0);

BEGIN
   PROCESS (clk, reset)
   BEGIN
      IF reset = '0' THEN
         state_reg <= idle;
         t0_reg <= (OTHERS => '0');
         t1_reg <= (OTHERS => '0');
         n_reg <= (OTHERS => '0');
      ELSIF (clk'event AND clk = '1') THEN
         state_reg <= state_next;
         t0_reg <= t0_next;
         t1_reg <= t1_next;
         n_reg <= n_next;
      END IF;

   END PROCESS;

   PROCESS (state_reg, n_reg, t0_reg, t1_reg, start, i, n_next)
   BEGIN
      ready <= '0';
      done_tick <= '0';
      CASE state_reg IS
         WHEN idle =>
            ready <= '1';
            IF start = '1' THEN
               t0_next <= (OTHERS => '0');
               t1_next <= (0 => '1', OTHERS => '0');
               n_next <= unsigned(i);
               state_next <= op;
            END IF;
         WHEN op =>
            IF n_reg = 0 THEN
               t1_next <= (OTHERS => '0');
               state_next <= done;
            ELSIF n_reg = 1 THEN
               state_next <= done;
            ELSE
               t1_next <= t1_reg + t0_reg;
               t0_next <= t1_reg;
               n_next <= n_reg - 1;
            END IF;
         WHEN done =>
            done_tick <= '1';
            state_next <= idle;
      END CASE;
   END PROCESS;

   f <= STD_LOGIC_VECTOR(t1_reg);

END arch;
```

### Testbench : `fib_tb`

Voici le testbench utilisé pour simuler l'entité `fib` :

```vhdl
LIBRARY ieee;
USE ieee.std_logic_1164.ALL;
USE ieee.numeric_std.ALL;

ENTITY fib_tb IS
END ENTITY;

ARCHITECTURE tb OF fib_tb IS

  COMPONENT fib IS
    PORT (
      clk, reset : IN STD_LOGIC;
      start : IN STD_LOGIC;
      i : IN STD_LOGIC_VECTOR(4 DOWNTO 0);
      ready : OUT STD_LOGIC;
      done_tick : OUT STD_LOGIC;
      f : OUT STD_LOGIC_VECTOR(19 DOWNTO 0)
    );
  END COMPONENT;
  SIGNAL rst_tb : STD_LOGIC;
  SIGNAL clk_tb : STD_LOGIC := '0';
  SIGNAL start_tb : STD_LOGIC := '1';

  SIGNAL i_tb : STD_LOGIC_VECTOR(4 DOWNTO 0) := "00110";
  SIGNAL ready_tb : STD_LOGIC;
  SIGNAL done_tb : STD_LOGIC;
  SIGNAL f_tb : STD_LOGIC_VECTOR(19 DOWNTO 0);

  CONSTANT hp : TIME := 50 ns;
BEGIN

  DUT : fib
  PORT MAP(
    clk => clk_tb,
    reset => rst_tb,
    start => start_tb,
    i => i_tb,
    ready => ready_tb,
    done_tick => done_tb,
    f => f_tb
  );

  clk_tb <= NOT clk_tb AFTER hp;
  rst_tb <= '0', '1' AFTER 2 * hp;

END tb;
```

## Simulation avec ModelSIM

### Chronogramme

Le chronogramme montre l'évolution des signaux dans le temps lors de la simulation :

- **clk** : Horloge, utilisée pour synchroniser les changements d'état.
- **reset** : Signal de réinitialisation.
- **start** : Indicateur pour commencer le calcul de Fibonacci.
- **ready** : Indicateur que le module est prêt à commencer.
- **done_tick** : Indicateur que le calcul est terminé.
- **f** : Résultat de Fibonacci en sortie.

![Capture d'écran 2024-12-14 151347](https://github.com/user-attachments/assets/afb98209-d26d-4c58-b3f9-4b7d1b77612a)


### Netlist

Une fois la simulation effectuée, vous pouvez générer la netlist à partir de Quartus Prime Lite pour vérifier la structure de votre conception. La netlist montre les connexions des composants et est utilisée pour la mise en œuvre du design sur du matériel.

![Capture d'écran 2024-12-14 152901](https://github.com/user-attachments/assets/aea8d79a-a173-4747-a96b-6a322f5bc42b)

## Conclusion

Ce laboratoire permet de comprendre la conception d'un calculateur de Fibonacci en utilisant VHDL et une machine à états. Grâce à la simulation, vous pouvez observer le comportement du module dans différents états et valider la fonctionnalité du calcul de Fibonacci.
