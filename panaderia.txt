'''Algoritmo panaderia de lamport'''
'''libreria que nos permite manejar hilos'''
import threading 
import time

'''el cliente es un hilo'''


'''tiempo de espera en cada seccion'''
DELAY=1 
'''Aqui podemos modificar el numero de hilos de nuestro programa'''
NUM_HILOS = 2
'''calcular el numero de turno'''
eligiendo=[False]*NUM_HILOS
numero= [0]*NUM_HILOS
critical=[False]*NUM_HILOS #se usa para monitorear que threads estan en la seccion critica en un momento dado

'''Cuando un hilo quiere entrar en su sección crítica, primero obtiene su número de turno, que calcula como el máximo de los turnos de los otros hilos, más uno.'''
def maximo():
    max = -1
    for i in numero:
        if (i>max):
            max=i
    
    return max

'''hace la comparacion de tuplas'''
'''
a<c : el numero de turno del hilo j es menor que el turno del hilo i
a==c: se puede dar la casualidad de que ambos hilos tengan el mismo turno para eso la siguiente condicion 
b<d : si dos o más hilos tienen el mismo número de turno, tiene más prioridad el hilo que tenga el identificador(son unicos) con un número más bajo
'''
def comparacion_tuplas(a,b,c,d):
    if(a<c):
        return True
    elif (a==c and b<d):
        return True
    return False

def bloqueado(i):

    eligiendo[i] = True
    numero[i]= 1 + maximo()
    eligiendo[i]= False

    
    for j in range(NUM_HILOS):

        '''se espera a que haya elegido'''
        while eligiendo[j]:
            continue
        while ((numero[j] != 0) and (comparacion_tuplas(numero[j],j,numero[i],i))):
            continue
        '''
        (a, b) < (c, d)
        es equivalente a 
        (a < c) o ((a == c) y (b < d))
        mirar comparacion_tuplas
        
        '''

def desbloqueado(i):
    '''esto dice que el hilo no quiere entrar a la seccion critica (mirar el segundo while de bloqueado donde debe ser != 0 para poder entrar)'''
    numero[i]=0 

def ejecutar_hilo(i):
        while(True):
                '''entra a la seccion critica'''
                bloqueado(i)
                print("Thread "+str(i)+" en seccion critica\n")
                critical[i]=True
                time.sleep(DELAY)
                critical[i]=False
                '''sale de la seccion critica'''
                desbloqueado(i)
                print("Thread "+str(i)+" fuera de seccion critica\n")
                time.sleep(DELAY)
                


'''****monitorea en que seccion estan los threads********
*******True: en la seccion critica************
*******False: fuera de ella*************'''
def monitorear():
        tiempo=1
        while(True):
                print ("Tiempo: "+str(tiempo))
                '''solo puede haber un True en el vector que se imprime
                porque solo hay un thread en la seccion critica a la vez'''
                print(critical)
                tiempo+=1
                time.sleep(1)




'''
Asi podemos mostrar el nombre y el identificador del hilo en el que estamos
threading.current_thread().getName()
threading.current_thread().ident'''

def panaderia():
        for num_hilo in range(NUM_HILOS):
                hilo = threading.Thread(name='%s' %num_hilo, target=ejecutar_hilo, args=(num_hilo,))    
                hilo.start()



panaderia()
monitorear()



