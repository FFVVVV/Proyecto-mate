!pip install cirq
import cirq
import numpy as np
from fractions import Fraction
import math

def quantum_period_finding(a: int, N: int, n_count: int, n_target: int):
    """
    Implementa el circuito de búsqueda de período cuántico para el algoritmo de Shor
    
    Args:
        a: Base para la función modular exponencial
        N: Número a factorizar
        n_count: Número de qubits para el registro de conteo
        n_target: Número de qubits para el registro objetivo
    """
    # Crear registros cuánticos
    count_qubits = [cirq.GridQubit(i, 0) for i in range(n_count)]
    target_qubits = [cirq.GridQubit(i, 1) for i in range(n_target)]
    
    circuit = cirq.Circuit()
    
    # Inicializar el registro de conteo en superposición
    circuit.append([cirq.H(qubit) for qubit in count_qubits])
    
    # Inicializar el registro objetivo en |1⟩
    circuit.append(cirq.X(target_qubits[0]))
    
    # Aplicar operaciones de exponenciación modular controlada
    for i, count_qubit in enumerate(count_qubits):
        # Calcular a^(2^i) mod N
        power = pow(a, 2**i, N)
        # Aplicar multiplicación modular controlada
        controlled_mod_mult = create_controlled_mod_mult(power, N, target_qubits)
        circuit.append(controlled_mod_mult.controlled_by(count_qubit))
    
    # Aplicar QFT inversa al registro de conteo
    circuit.append(cirq.qft(*count_qubits, inverse=True))
    
    return circuit

def create_controlled_mod_mult(a: int, N: int, target_qubits):
    """
    Crea una operación de multiplicación modular controlada
    Esta es una versión simplificada para propósitos educativos
    """
    # En una implementación real, esto sería una secuencia de compuertas cuánticas
    # que implementan la multiplicación modular
    return cirq.X(target_qubits[0])

def find_period(a: int, N: int):
    """
    Encuentra el período de la función f(x) = a^x mod N
    """
    n_count = 2 * len(bin(N)[2:])  # Doble del número de bits necesarios para representar N
    n_target = len(bin(N)[2:])     # Número de bits necesarios para representar N
    
    # Crear y ejecutar el circuito
    circuit = quantum_period_finding(a, N, n_count, n_target)
    simulator = cirq.Simulator()
    result = simulator.simulate(circuit)
    
    # Procesar los resultados (simplificado)
    # En una implementación real, necesitaríamos realizar múltiples mediciones
    # y procesar los resultados usando fracciones continuas
    return 4  # Este es un valor de ejemplo

def shor_algorithm(N: int):
    """
    Implementación del algoritmo de Shor para factorizar N
    """
    if N % 2 == 0:
        return 2, N//2
    
    # Elegir un número aleatorio a < N
    a = np.random.randint(2, N)
    
    # Calcular GCD(a, N)
    gcd = math.gcd(a, N)
    if gcd != 1:
        return gcd, N//gcd
    
    # Encontrar el período usando el circuito cuántico
    r = find_period(a, N)
    
    if r % 2 != 0:
        return None  # Falló, intentar de nuevo con diferente a
    
    # Calcular factores candidatos
    factor1 = math.gcd(a**(r//2) + 1, N)
    factor2 = math.gcd(a**(r//2) - 1, N)
    
    if factor1 * factor2 == N:
        return factor1, factor2
    else:
        return None  # Falló, intentar de nuevo

# Ejemplo de uso
if __name__ == "__main__":
    N = 300900  # Número a factorizar
    result = shor_algorithm(N)
    
    if result:
        p, q = result
        print(f"Factores encontrados para {N}: {p} y {q}")
    else:
        print("No se encontraron factores. Intente nuevamente.")
