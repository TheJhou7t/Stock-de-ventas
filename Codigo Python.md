import datetime
import json
import os

# Archivos de persistencia
DATA_DIR = "data"
CLIENTES_FILE = f"{DATA_DIR}/clientes.json"
VENTAS_FILE = f"{DATA_DIR}/ventas.json"
INVENTARIO_FILE = f"{DATA_DIR}/inventario.json"
CUENTAS_FILE = f"{DATA_DIR}/cuentas_por_cobrar.json"

# Clase para manejar la persistencia de datos
class DataManager:
    @staticmethod
    def cargar_datos(file_path, default_data):
        if os.path.exists(file_path):
            with open(file_path, "r") as file:
                return json.load(file)
        else:
            return default_data

    @staticmethod
    def guardar_datos(file_path, data):
        os.makedirs(DATA_DIR, exist_ok=True)
        with open(file_path, "w") as file:
            json.dump(data, file, indent=4)

# Clase para gestionar el negocio
class SistemaGestion:
    def __init__(self):
        # Cargar datos desde los archivos
        self.clientes = DataManager.cargar_datos(CLIENTES_FILE, {})
        self.ventas = DataManager.cargar_datos(VENTAS_FILE, [])
        self.inventario = DataManager.cargar_datos(INVENTARIO_FILE, {})
        self.cuentas_por_cobrar = DataManager.cargar_datos(CUENTAS_FILE, [])

    def guardar_todo(self):
        DataManager.guardar_datos(CLIENTES_FILE, self.clientes)
        DataManager.guardar_datos(VENTAS_FILE, self.ventas)
        DataManager.guardar_datos(INVENTARIO_FILE, self.inventario)
        DataManager.guardar_datos(CUENTAS_FILE, self.cuentas_por_cobrar)

    def registrar_venta(self):
        print("\n--- Registrar Venta ---")
        cliente = input("Ingrese el nombre del cliente: ")
        if cliente not in self.clientes:
            self.clientes[cliente] = {"historial_compras": []}

        producto = input("Ingrese el producto vendido: ")
        if producto not in self.inventario or self.inventario[producto]["cantidad"] == 0:
            print("Producto no disponible en el inventario.")
            return

        cantidad = int(input(f"Ingrese la cantidad de {producto}: "))
        if cantidad > self.inventario[producto]["cantidad"]:
            print("Stock insuficiente.")
            return

        precio = self.inventario[producto]["precio"]
        total = cantidad * precio
        metodo_pago = input("Método de pago (contado/fiado): ").lower()
        fecha = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")

        # Registrar la venta
        venta = {
            "cliente": cliente,
            "producto": producto,
            "cantidad": cantidad,
            "total": total,
            "fecha": fecha,
            "metodo_pago": metodo_pago,
        }
        self.ventas.append(venta)
        self.clientes[cliente]["historial_compras"].append(venta)

        # Actualizar inventario
        self.inventario[producto]["cantidad"] -= cantidad

        # Registrar cuentas por cobrar si es fiado
        if metodo_pago == "fiado":
            self.cuentas_por_cobrar.append({"cliente": cliente, "total": total, "fecha": fecha})

        print(f"Venta registrada exitosamente. Total: ${total:.2f}")
        self.guardar_todo()

    def actualizar_inventario(self):
        print("\n--- Actualizar Inventario ---")
        producto = input("Ingrese el nombre del producto: ")
        cantidad = int(input(f"Ingrese la cantidad para {producto}: "))
        precio = float(input(f"Ingrese el precio unitario de {producto}: "))

        if producto in self.inventario:
            self.inventario[producto]["cantidad"] += cantidad
        else:
            self.inventario[producto] = {"cantidad": cantidad, "precio": precio}

        print("Inventario actualizado correctamente.")
        self.guardar_todo()

    def gestionar_cuentas_por_cobrar(self):
        print("\n--- Cuentas por Cobrar ---")
        if not self.cuentas_por_cobrar:
            print("No hay cuentas pendientes.")
            return

        for deuda in self.cuentas_por_cobrar:
            print(f"Cliente: {deuda['cliente']}, Total: ${deuda['total']:.2f}, Fecha: {deuda['fecha']}")

        cliente = input("Ingrese el nombre del cliente para registrar el pago: ")
        monto_pagado = float(input("Ingrese el monto pagado: "))

        for deuda in self.cuentas_por_cobrar:
            if deuda["cliente"] == cliente:
                deuda["total"] -= monto_pagado
                if deuda["total"] <= 0:
                    self.cuentas_por_cobrar.remove(deuda)
                print("Pago registrado correctamente.")
                self.guardar_todo()
                return

        print("Cliente no encontrado.")

    def generar_reporte(self):
        print("\n--- Generar Reporte ---")
        print("1. Ventas")
        print("2. Inventario")
        opcion = int(input("Seleccione el tipo de reporte: "))

        if opcion == 1:
            if not self.ventas:
                print("No hay ventas registradas.")
                return
            for venta in self.ventas:
                print(f"{venta}")
        elif opcion == 2:
            if not self.inventario:
                print("Inventario vacío.")
                return
            for producto, detalles in self.inventario.items():
                print(f"{producto}: {detalles}")
        else:
            print("Opción inválida.")

# Menú Principal
def menu():
    sistema = SistemaGestion()
    while True:
        print("\n--- Sistema de Gestión de Ventas e Inventario ---")
        print("1. Registrar Venta")
        print("2. Actualizar Inventario")
        print("3. Gestionar Cuentas por Cobrar")
        print("4. Generar Reportes")
        print("5. Salir")
        opcion = input("Seleccione una opción: ")
        if opcion == "1":
            sistema.registrar_venta()
        elif opcion == "2":
            sistema.actualizar_inventario()
        elif opcion == "3":
            sistema.gestionar_cuentas_por_cobrar()
        elif opcion == "4":
            sistema.generar_reporte()
        elif opcion == "5":
            print("Saliendo del sistema...")
            break
        else:
            print("Opción inválida.")

# Iniciar el programa
if __name__ == "__main__":
    menu()
