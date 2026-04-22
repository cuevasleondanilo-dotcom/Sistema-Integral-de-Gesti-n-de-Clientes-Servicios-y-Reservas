# Sistema-Integral-de-Gesti-n-de-Clientes-Servicios-y-Reservas
"""
Sistema Integral de Gestión de Clientes, Servicios y Reservas - Software FJ
Con Interfaz Gráfica - VERSIÓN CORREGIDA CON ELIMINACIÓN DE CLIENTES
"""

import tkinter as tk
from tkinter import ttk, messagebox, scrolledtext, simpledialog
import datetime
import os
import traceback
from abc import ABC, abstractmethod
from typing import List, Dict, Optional
from enum import Enum


# ==================== ARCHIVO DE LOGS ====================
class Logger:
    """Clase para manejar el registro de eventos y errores"""
    
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._initialized = False
        return cls._instance
    
    def __init__(self):
        if hasattr(self, '_initialized') and self._initialized:
            return
        self._initialized = True
        self.log_file = "sistema_fj.log"
        self.error_file = "errores_fj.log"
        self._crear_archivos()
    
    def _crear_archivos(self):
        """Crea los archivos de log si no existen"""
        try:
            if not os.path.exists(self.log_file):
                with open(self.log_file, 'w', encoding='utf-8') as f:
                    f.write(f"{'='*80}\n")
                    f.write(f"LOG DEL SISTEMA - SOFTWARE FJ\n")
                    f.write(f"Iniciado: {datetime.datetime.now()}\n")
                    f.write(f"{'='*80}\n\n")
            
            if not os.path.exists(self.error_file):
                with open(self.error_file, 'w', encoding='utf-8') as f:
                    f.write(f"{'='*80}\n")
                    f.write(f"REGISTRO DE ERRORES - SOFTWARE FJ\n")
                    f.write(f"Iniciado: {datetime.datetime.now()}\n")
                    f.write(f"{'='*80}\n\n")
        except Exception:
            pass
    
    def log_evento(self, mensaje, nivel="INFO"):
        """Registra un evento en el archivo de log"""
        timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        try:
            with open(self.log_file, 'a', encoding='utf-8') as f:
                f.write(f"[{timestamp}] [{nivel}] {mensaje}\n")
        except Exception:
            pass
    
    def log_error(self, error, contexto=""):
        """Registra un error en el archivo de errores"""
        timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        try:
            with open(self.error_file, 'a', encoding='utf-8') as f:
                f.write(f"\n{'='*60}\n")
                f.write(f"[{timestamp}] ERROR: {type(error).__name__}\n")
                f.write(f"Contexto: {contexto}\n")
                f.write(f"Mensaje: {str(error)}\n")
                f.write(f"{'='*60}\n")
        except Exception:
            pass


# ==================== EXCEPCIONES PERSONALIZADAS ====================
class SistemaFJError(Exception):
    pass

class ClienteInvalidoError(SistemaFJError):
    pass

class ServicioNoDisponibleError(SistemaFJError):
    pass

class ReservaInvalidaError(SistemaFJError):
    pass

class ParametroInvalidoError(SistemaFJError):
    pass


# ==================== CLASE ABSTRACTA ENTIDAD ====================
class Entidad(ABC):
    def __init__(self, id_entidad):
        self._id_entidad = id_entidad
        self._fecha_creacion = datetime.datetime.now()
    
    @abstractmethod
    def validar(self):
        pass
    
    @abstractmethod
    def __str__(self):
        pass
    
    def get_id(self):
        return self._id_entidad


# ==================== CLASE CLIENTE ====================
class Cliente(Entidad):
    _clientes_registrados = set()
    
    def __init__(self, id_cliente, nombre, email, telefono, direccion="", vip=False):
        super().__init__(id_cliente)
        self._nombre = nombre
        self._email = email
        self._telefono = telefono
        self._direccion = direccion
        self._vip = vip
        self._reservas_realizadas = []
        self._total_gastado = 0.0
        
        if not self.validar():
            raise ClienteInvalidoError(f"Cliente {id_cliente} no pasó validaciones")
        
        if id_cliente in Cliente._clientes_registrados:
            raise ClienteInvalidoError(f"Ya existe cliente con ID {id_cliente}")
        
        Cliente._clientes_registrados.add(id_cliente)
    
    def validar(self):
        try:
            if not self._id_entidad or len(self._id_entidad) < 3:
                return False
            if not self._nombre or len(self._nombre.strip()) < 2:
                return False
            if not self._email or '@' not in self._email or '.' not in self._email:
                return False
            if not self._telefono or len(self._telefono) < 7:
                return False
            return True
        except Exception:
            return False
    
    def get_nombre(self):
        return self._nombre
    
    def get_email(self):
        return self._email
    
    def get_telefono(self):
        return self._telefono
    
    def get_direccion(self):
        return self._direccion
    
    def is_vip(self):
        return self._vip
    
    def set_vip(self, vip):
        self._vip = vip
    
    def agregar_reserva(self, reserva):
        self._reservas_realizadas.append(reserva)
    
    def agregar_gasto(self, monto):
        self._total_gastado += monto
    
    def get_total_gastado(self):
        return self._total_gastado
    
    def get_reservas(self):
        return self._reservas_realizadas.copy()
    
    def __str__(self):
        return f"Cliente: {self._nombre} (ID: {self._id_entidad}) - VIP: {self._vip}"


# ==================== CLASE ABSTRACTA SERVICIO ====================
class Servicio(Entidad):
    def __init__(self, id_servicio, nombre, precio_base, duracion_base):
        super().__init__(id_servicio)
        self._nombre = nombre
        self._precio_base = precio_base
        self._duracion_base = duracion_base
        self._disponible = True
    
    @abstractmethod
    def calcular_costo(self, duracion=None, **kwargs):
        pass
    
    @abstractmethod
    def describir(self):
        pass
    
    @abstractmethod
    def validar_parametros(self, **kwargs):
        pass
    
    def get_nombre(self):
        return self._nombre
    
    def get_precio_base(self):
        return self._precio_base
    
    def get_duracion_base(self):
        return self._duracion_base
    
    def esta_disponible(self):
        return self._disponible
    
    def set_disponible(self, disponible):
        self._disponible = disponible
    
    def validar(self):
        return (self._id_entidad and self._nombre and self._precio_base > 0 and self._duracion_base > 0)
    
    def __str__(self):
        return f"Servicio: {self._nombre} - ${self._precio_base:,.2f}"


# ==================== SERVICIOS ESPECIALIZADOS ====================
class ReservaSalas(Servicio):
    _CAPACIDADES_SALA = {"pequeña": 10, "mediana": 25, "grande": 50, "ejecutiva": 100}
    
    def __init__(self, id_servicio, nombre, precio_base, duracion_base, tipo_sala="mediana"):
        super().__init__(id_servicio, nombre, precio_base, duracion_base)
        self._tipo_sala = tipo_sala
        self._capacidad = self._CAPACIDADES_SALA.get(tipo_sala, 25)
    
    def calcular_costo(self, duracion=None, **kwargs):
        duracion_efectiva = duracion if duracion else self._duracion_base
        costo = self._precio_base * (duracion_efectiva / self._duracion_base)
        factores = {"pequeña": 0.8, "mediana": 1.0, "grande": 1.5, "ejecutiva": 2.0}
        costo *= factores.get(self._tipo_sala, 1.0)
        
        if 'descuento' in kwargs:
            descuento = kwargs['descuento']
            if 0 <= descuento <= 50:
                costo *= (1 - descuento / 100)
        
        if 'impuesto' in kwargs:
            impuesto = kwargs['impuesto']
            if 0 <= impuesto <= 30:
                costo *= (1 + impuesto / 100)
        
        if 'servicios_extra' in kwargs:
            costo += kwargs['servicios_extra'] * 50000
        
        return round(costo, 2)
    
    def calcular_costo_estandar(self, duracion=None):
        return self.calcular_costo(duracion)
    
    def describir(self):
        return f"Reserva de sala {self._tipo_sala} - Capacidad: {self._capacidad} personas"
    
    def validar_parametros(self, **kwargs):
        if 'descuento' in kwargs and (kwargs['descuento'] < 0 or kwargs['descuento'] > 50):
            raise ParametroInvalidoError("El descuento debe estar entre 0 y 50%")
        return True
    
    def get_capacidad(self):
        return self._capacidad
    
    def get_tipo_sala(self):
        return self._tipo_sala


class AlquilerEquipos(Servicio):
    _EQUIPOS_DISPONIBLES = ["proyector", "computador", "pantalla", "sonido", "videoconferencia"]
    
    def __init__(self, id_servicio, nombre, precio_base, duracion_base, equipos=None):
        super().__init__(id_servicio, nombre, precio_base, duracion_base)
        self._equipos = equipos if equipos else []
        self._costo_adicional_equipo = 20000
    
    def agregar_equipo(self, equipo):
        if equipo in self._EQUIPOS_DISPONIBLES:
            self._equipos.append(equipo)
        else:
            raise ServicioNoDisponibleError(f"Equipo '{equipo}' no disponible")
    
    def calcular_costo(self, duracion=None, **kwargs):
        duracion_efectiva = duracion if duracion else self._duracion_base
        costo = self._precio_base * (duracion_efectiva / self._duracion_base)
        costo += len(self._equipos) * self._costo_adicional_equipo
        
        if 'descuento_largo' in kwargs and kwargs['descuento_largo']:
            if duracion_efectiva > 480:
                costo *= 0.85
        
        return round(costo, 2)
    
    def describir(self):
        equipos_str = ", ".join(self._equipos) if self._equipos else "Ninguno"
        return f"Alquiler de equipos - Equipos: {equipos_str}"
    
    def validar_parametros(self, **kwargs):
        return True
    
    def get_equipos(self):
        return self._equipos.copy()


class AsesoriaEspecializada(Servicio):
    _AREAS = ["tecnologia", "negocios", "marketing", "finanzas", "recursos_humanos"]
    
    def __init__(self, id_servicio, nombre, precio_base, duracion_base, area="tecnologia", nivel_experto="senior"):
        super().__init__(id_servicio, nombre, precio_base, duracion_base)
        self._area = area
        self._nivel_experto = nivel_experto
        self._asesor_asignado = None
    
    def calcular_costo(self, duracion=None, **kwargs):
        duracion_efectiva = duracion if duracion else self._duracion_base
        costo = self._precio_base * (duracion_efectiva / 60)
        factores = {"junior": 1.0, "semisenior": 1.5, "senior": 2.0, "expert": 3.0}
        costo *= factores.get(self._nivel_experto, 1.0)
        
        if 'vip' in kwargs and kwargs['vip']:
            costo *= 0.9
        
        return round(costo, 2)
    
    def describir(self):
        return f"Asesoría en {self._area} - Nivel {self._nivel_experto}"
    
    def validar_parametros(self, **kwargs):
        return True
    
    def asignar_asesor(self, asesor):
        self._asesor_asignado = asesor
    
    def get_area(self):
        return self._area


# ==================== CLASE RESERVA ====================
class EstadoReserva(Enum):
    PENDIENTE = "Pendiente"
    CONFIRMADA = "Confirmada"
    CANCELADA = "Cancelada"
    COMPLETADA = "Completada"
    EN_CURSO = "En Curso"


class Reserva:
    def __init__(self, id_reserva, cliente, servicio, fecha, duracion, parametros_extra=None):
        self._id_reserva = id_reserva
        self._cliente = cliente
        self._servicio = servicio
        self._fecha = fecha
        self._duracion = duracion
        self._parametros_extra = parametros_extra if parametros_extra else {}
        self._estado = EstadoReserva.PENDIENTE
        self._costo_total = 0.0
        self._fecha_reserva = datetime.datetime.now()
        self._validar_reserva()
    
    def _validar_reserva(self):
        if not self._cliente:
            raise ReservaInvalidaError("Cliente no válido")
        if not self._servicio:
            raise ReservaInvalidaError("Servicio no válido")
        if not self._servicio.esta_disponible():
            raise ServicioNoDisponibleError(f"Servicio '{self._servicio.get_nombre()}' no disponible")
        if self._duracion < 30:
            raise ReservaInvalidaError("La duración mínima es 30 minutos")
        if self._duracion > 1440:
            raise ReservaInvalidaError("La duración máxima es 24 horas")
    
    def confirmar(self):
        try:
            if self._estado == EstadoReserva.PENDIENTE:
                if not self._servicio.esta_disponible():
                    raise ServicioNoDisponibleError("El servicio ya no está disponible")
                
                self._costo_total = self._servicio.calcular_costo(
                    duracion=self._duracion,
                    **self._parametros_extra
                )
                
                self._estado = EstadoReserva.CONFIRMADA
                self._cliente.agregar_reserva(self)
                self._cliente.agregar_gasto(self._costo_total)
                
                logger = Logger()
                logger.log_evento(f"Reserva {self._id_reserva} confirmada para {self._cliente.get_nombre()}")
                return True
            return False
        except Exception as e:
            logger = Logger()
            logger.log_error(e, f"Confirmando reserva {self._id_reserva}")
            raise
    
    def cancelar(self, motivo=""):
        try:
            if self._estado in [EstadoReserva.PENDIENTE, EstadoReserva.CONFIRMADA]:
                self._estado = EstadoReserva.CANCELADA
                logger = Logger()
                logger.log_evento(f"Reserva {self._id_reserva} cancelada. Motivo: {motivo}")
                return True
            return False
        except Exception as e:
            logger = Logger()
            logger.log_error(e, f"Cancelando reserva {self._id_reserva}")
            raise
    
    def procesar(self):
        try:
            if self._estado == EstadoReserva.CONFIRMADA:
                self._estado = EstadoReserva.EN_CURSO
                logger = Logger()
                logger.log_evento(f"Reserva {self._id_reserva} en curso")
                return True
            return False
        except Exception as e:
            logger = Logger()
            logger.log_error(e, f"Procesando reserva {self._id_reserva}")
            raise
    
    def completar(self):
        try:
            if self._estado == EstadoReserva.EN_CURSO:
                self._estado = EstadoReserva.COMPLETADA
                logger = Logger()
                logger.log_evento(f"Reserva {self._id_reserva} completada")
                return True
            return False
        except Exception as e:
            logger = Logger()
            logger.log_error(e, f"Completando reserva {self._id_reserva}")
            raise
    
    def mostrar_info(self, detallado=False):
        info = f"Reserva: {self._id_reserva}\n"
        info += f"Cliente: {self._cliente.get_nombre()}\n"
        info += f"Servicio: {self._servicio.get_nombre()}\n"
        info += f"Estado: {self._estado.value}\n"
        
        if detallado:
            info += f"Fecha: {self._fecha.strftime('%Y-%m-%d %H:%M')}\n"
            info += f"Duración: {self._duracion} minutos\n"
            info += f"Costo: ${self._costo_total:,.2f}\n"
            info += f"Descripción: {self._servicio.describir()}\n"
        
        return info
    
    def __str__(self):
        return self.mostrar_info(False)
    
    def get_id(self):
        return self._id_reserva
    
    def get_estado(self):
        return self._estado
    
    def get_costo(self):
        return self._costo_total
    
    def get_cliente(self):
        return self._cliente
    
    def get_servicio(self):
        return self._servicio
    
    def get_fecha(self):
        return self._fecha
    
    def get_duracion(self):
        return self._duracion


# ==================== SISTEMA DE GESTIÓN ====================
class SistemaGestionFJ:
    def __init__(self):
        self._clientes = {}
        self._servicios = {}
        self._reservas = {}
        self._logger = Logger()
        self._logger.log_evento("Sistema iniciado correctamente")
    
    def registrar_cliente(self, id_cliente, nombre, email, telefono, direccion="", vip=False):
        try:
            cliente = Cliente(id_cliente, nombre, email, telefono, direccion, vip)
            self._clientes[id_cliente] = cliente
            self._logger.log_evento(f"Cliente registrado: {nombre}")
            return cliente
        except ClienteInvalidoError as e:
            self._logger.log_error(e, f"Registrando cliente {id_cliente}")
            messagebox.showerror("Error", str(e))
            return None
        except Exception as e:
            self._logger.log_error(e, f"Error registrando cliente {id_cliente}")
            messagebox.showerror("Error", f"Error inesperado: {str(e)}")
            return None
    
    def buscar_cliente(self, id_cliente):
        try:
            return self._clientes.get(id_cliente)
        except Exception as e:
            self._logger.log_error(e, f"Buscando cliente {id_cliente}")
            return None
    
    def eliminar_cliente(self, id_cliente):
        """Elimina un cliente del sistema"""
        try:
            if id_cliente not in self._clientes:
                messagebox.showwarning("No encontrado", f"No existe cliente con ID {id_cliente}")
                return False
            
            cliente = self._clientes[id_cliente]
            
            # Verificar si tiene reservas activas (pendientes o confirmadas)
            reservas_activas = []
            for reserva in self._reservas.values():
                if reserva.get_cliente().get_id() == id_cliente:
                    estado = reserva.get_estado()
                    if estado in [EstadoReserva.PENDIENTE, EstadoReserva.CONFIRMADA, EstadoReserva.EN_CURSO]:
                        reservas_activas.append(reserva)
            
            if reservas_activas:
                mensaje = f"El cliente tiene {len(reservas_activas)} reservas activas:\n"
                for r in reservas_activas:
                    mensaje += f"  - {r.get_id()}: {r.get_estado().value}\n"
                mensaje += "\n¿Desea eliminarlo de todas formas? (Esto cancelará todas sus reservas)"
                
                if not messagebox.askyesno("Confirmar", mensaje):
                    return False
                
                # Cancelar todas las reservas activas
                for reserva in reservas_activas:
                    reserva.cancelar("Cliente eliminado del sistema")
            
            # Eliminar el cliente
            del self._clientes[id_cliente]
            self._logger.log_evento(f"Cliente eliminado: {id_cliente}")
            messagebox.showinfo("Éxito", f"Cliente {id_cliente} eliminado correctamente")
            return True
            
        except Exception as e:
            self._logger.log_error(e, f"Eliminando cliente {id_cliente}")
            messagebox.showerror("Error", f"Error al eliminar cliente: {str(e)}")
            return False
    
    def agregar_servicio(self, servicio):
        try:
            if not servicio.validar():
                raise ServicioNoDisponibleError("Servicio inválido")
            self._servicios[servicio.get_id()] = servicio
            self._logger.log_evento(f"Servicio agregado: {servicio.get_nombre()}")
            return True
        except Exception as e:
            self._logger.log_error(e, f"Agregando servicio {servicio.get_id()}")
            return False
    
    def buscar_servicio(self, id_servicio):
        try:
            return self._servicios.get(id_servicio)
        except Exception as e:
            self._logger.log_error(e, f"Buscando servicio {id_servicio}")
            return None
    
    def listar_servicios_disponibles(self):
        return [s for s in self._servicios.values() if s.esta_disponible()]
    
    def listar_clientes(self):
        return list(self._clientes.values())
    
    def listar_reservas(self):
        return list(self._reservas.values())
    
    def crear_reserva(self, id_reserva, id_cliente, id_servicio, fecha_str, duracion, parametros_extra=None):
        try:
            cliente = self.buscar_cliente(id_cliente)
            if not cliente:
                raise ReservaInvalidaError(f"Cliente {id_cliente} no encontrado")
            
            servicio = self.buscar_servicio(id_servicio)
            if not servicio:
                raise ReservaInvalidaError(f"Servicio {id_servicio} no encontrado")
            
            if not servicio.esta_disponible():
                raise ServicioNoDisponibleError(f"Servicio {servicio.get_nombre()} no disponible")
            
            try:
                fecha = datetime.datetime.strptime(fecha_str, "%Y-%m-%d %H:%M")
            except ValueError:
                raise ParametroInvalidoError("Formato de fecha inválido. Use: YYYY-MM-DD HH:MM")
            
            reserva = Reserva(id_reserva, cliente, servicio, fecha, duracion, parametros_extra)
            
            # Confirmar automáticamente si es VIP
            if cliente.is_vip():
                reserva.confirmar()
            
            self._reservas[id_reserva] = reserva
            self._logger.log_evento(f"Reserva creada: {id_reserva} para {cliente.get_nombre()}")
            return reserva
            
        except (ReservaInvalidaError, ServicioNoDisponibleError, ParametroInvalidoError) as e:
            self._logger.log_error(e, f"Creando reserva {id_reserva}")
            messagebox.showerror("Error", str(e))
            return None
        except Exception as e:
            self._logger.log_error(e, f"Error creando reserva {id_reserva}")
            messagebox.showerror("Error", f"Error inesperado: {str(e)}")
            return None
    
    def confirmar_reserva(self, id_reserva):
        try:
            reserva = self._reservas.get(id_reserva)
            if not reserva:
                raise ReservaInvalidaError(f"Reserva {id_reserva} no encontrada")
            resultado = reserva.confirmar()
            if resultado:
                self._logger.log_evento(f"Reserva {id_reserva} confirmada")
            return resultado
        except Exception as e:
            self._logger.log_error(e, f"Confirmando reserva {id_reserva}")
            messagebox.showerror("Error", str(e))
            return False
    
    def cancelar_reserva(self, id_reserva, motivo=""):
        try:
            reserva = self._reservas.get(id_reserva)
            if not reserva:
                raise ReservaInvalidaError(f"Reserva {id_reserva} no encontrada")
            resultado = reserva.cancelar(motivo)
            if resultado:
                self._logger.log_evento(f"Reserva {id_reserva} cancelada")
            return resultado
        except Exception as e:
            self._logger.log_error(e, f"Cancelando reserva {id_reserva}")
            messagebox.showerror("Error", str(e))
            return False
    
    def procesar_reserva(self, id_reserva):
        try:
            reserva = self._reservas.get(id_reserva)
            if not reserva:
                raise ReservaInvalidaError(f"Reserva {id_reserva} no encontrada")
            resultado = reserva.procesar()
            if resultado:
                self._logger.log_evento(f"Reserva {id_reserva} procesada/iniciada")
            return resultado
        except Exception as e:
            self._logger.log_error(e, f"Procesando reserva {id_reserva}")
            messagebox.showerror("Error", str(e))
            return False
    
    def completar_reserva(self, id_reserva):
        try:
            reserva = self._reservas.get(id_reserva)
            if not reserva:
                raise ReservaInvalidaError(f"Reserva {id_reserva} no encontrada")
            resultado = reserva.completar()
            if resultado:
                self._logger.log_evento(f"Reserva {id_reserva} completada")
            return resultado
        except Exception as e:
            self._logger.log_error(e, f"Completando reserva {id_reserva}")
            messagebox.showerror("Error", str(e))
            return False
    
    def generar_reporte_ingresos(self):
        total_ingresos = 0
        ingresos_por_servicio = {}
        reservas_completadas = 0
        
        for reserva in self._reservas.values():
            if reserva.get_estado() == EstadoReserva.COMPLETADA:
                costo = reserva.get_costo()
                total_ingresos += costo
                reservas_completadas += 1
                
                servicio_nombre = reserva.get_servicio().get_nombre()
                if servicio_nombre in ingresos_por_servicio:
                    ingresos_por_servicio[servicio_nombre] += costo
                else:
                    ingresos_por_servicio[servicio_nombre] = costo
        
        return {
            "total_ingresos": total_ingresos,
            "ingresos_por_servicio": ingresos_por_servicio,
            "total_reservas_completadas": reservas_completadas
        }
    
    def obtener_reserva_por_id(self, id_reserva):
        return self._reservas.get(id_reserva)


# ==================== INTERFAZ GRÁFICA ====================
class AplicacionFJ:
    def __init__(self, root):
        self.root = root
        self.root.title("Software FJ - Sistema de Gestión de Clientes y Reservas")
        self.root.geometry("1400x800")
        self.root.configure(bg='#f0f0f0')
        
        self.sistema = SistemaGestionFJ()
        
        # Inicializar servicios por defecto
        self._inicializar_servicios()
        
        self._crear_interfaz()
        self._actualizar_listas()
    
    def _inicializar_servicios(self):
        """Crear servicios de ejemplo"""
        servicio1 = ReservaSalas("SERV001", "Reserva Sala Ejecutiva", 200000, 120, "ejecutiva")
        servicio2 = AlquilerEquipos("SERV002", "Alquiler Equipos Premium", 150000, 240, ["proyector", "sonido"])
        servicio3 = AsesoriaEspecializada("SERV003", "Asesoría Tecnológica", 100000, 60, "tecnologia", "senior")
        servicio4 = ReservaSalas("SERV004", "Reserva Sala Pequeña", 80000, 120, "pequeña")
        servicio5 = AlquilerEquipos("SERV005", "Alquiler Básico", 80000, 120, ["proyector"])
        
        self.sistema.agregar_servicio(servicio1)
        self.sistema.agregar_servicio(servicio2)
        self.sistema.agregar_servicio(servicio3)
        self.sistema.agregar_servicio(servicio4)
        self.sistema.agregar_servicio(servicio5)
    
    def _crear_interfaz(self):
        # Frame principal
        main_frame = ttk.Frame(self.root)
        main_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)
        
        # Título
        title_frame = ttk.Frame(main_frame)
        title_frame.pack(fill=tk.X, pady=(0, 10))
        
        ttk.Label(title_frame, text="SOFTWARE FJ", font=('Arial', 24, 'bold')).pack()
        ttk.Label(title_frame, text="Sistema Integral de Gestión de Clientes, Servicios y Reservas", 
                 font=('Arial', 12)).pack()
        
        # Separador
        ttk.Separator(main_frame, orient='horizontal').pack(fill=tk.X, pady=10)
        
        # Panel de pestañas
        self.notebook = ttk.Notebook(main_frame)
        self.notebook.pack(fill=tk.BOTH, expand=True)
        
        # Crear pestañas
        self._crear_pestana_clientes()
        self._crear_pestana_servicios()
        self._crear_pestana_reservas()
        self._crear_pestana_reportes()
    
    def _crear_pestana_clientes(self):
        tab = ttk.Frame(self.notebook)
        self.notebook.add(tab, text="👥 Clientes")
        
        # Panel izquierdo - Formulario
        left_frame = ttk.LabelFrame(tab, text="Registrar Cliente", padding=10)
        left_frame.pack(side=tk.LEFT, fill=tk.Y, padx=(0, 10))
        
        campos = [
            ("ID Cliente:", "id_cliente"),
            ("Nombre:", "nombre"),
            ("Email:", "email"),
            ("Teléfono:", "telefono"),
            ("Dirección:", "direccion")
        ]
        
        self.entries_clientes = {}
        for i, (label, key) in enumerate(campos):
            ttk.Label(left_frame, text=label).grid(row=i, column=0, sticky=tk.W, pady=5)
            entry = ttk.Entry(left_frame, width=25)
            entry.grid(row=i, column=1, pady=5, padx=(10, 0))
            self.entries_clientes[key] = entry
        
        self.vip_var = tk.BooleanVar()
        ttk.Checkbutton(left_frame, text="Cliente VIP", variable=self.vip_var).grid(row=len(campos), column=0, columnspan=2, pady=10)
        
        ttk.Button(left_frame, text="Registrar Cliente", command=self._registrar_cliente, width=30).grid(row=len(campos)+1, column=0, columnspan=2, pady=10)
        
        # Panel derecho - Lista de clientes
        right_frame = ttk.LabelFrame(tab, text="Lista de Clientes", padding=10)
        right_frame.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
        
        columns = ("ID", "Nombre", "Email", "Teléfono", "VIP", "Total Gastado")
        self.tree_clientes = ttk.Treeview(right_frame, columns=columns, show='headings', height=20)
        
        for col in columns:
            self.tree_clientes.heading(col, text=col)
            self.tree_clientes.column(col, width=120)
        
        scrollbar = ttk.Scrollbar(right_frame, orient=tk.VERTICAL, command=self.tree_clientes.yview)
        self.tree_clientes.configure(yscrollcommand=scrollbar.set)
        
        self.tree_clientes.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
        
        # Frame para ID de eliminación
        eliminar_frame = ttk.Frame(right_frame)
        eliminar_frame.pack(fill=tk.X, pady=(10, 5))
        
        ttk.Label(eliminar_frame, text="ID Cliente a eliminar:").pack(side=tk.LEFT, padx=(0, 10))
        self.eliminar_id_entry = ttk.Entry(eliminar_frame, width=15)
        self.eliminar_id_entry.pack(side=tk.LEFT, padx=(0, 10))
        
        # Botones de acción
        btn_frame = ttk.Frame(right_frame)
        btn_frame.pack(fill=tk.X, pady=5)
        
        ttk.Button(btn_frame, text="Eliminar Cliente", command=self._eliminar_cliente).pack(side=tk.LEFT, padx=5)
        ttk.Button(btn_frame, text="Eliminar Cliente Seleccionado", command=self._eliminar_cliente_seleccionado).pack(side=tk.LEFT, padx=5)
        ttk.Button(btn_frame, text="Actualizar Lista", command=self._actualizar_clientes).pack(side=tk.LEFT, padx=5)
        
        # Evento de selección
        self.tree_clientes.bind('<<TreeviewSelect>>', self._on_cliente_seleccionado)
    
    def _crear_pestana_servicios(self):
        tab = ttk.Frame(self.notebook)
        self.notebook.add(tab, text="🛠️ Servicios")
        
        # Lista de servicios
        frame = ttk.LabelFrame(tab, text="Servicios Disponibles", padding=10)
        frame.pack(fill=tk.BOTH, expand=True)
        
        columns = ("ID", "Nombre", "Precio Base", "Duración Base", "Descripción")
        self.tree_servicios = ttk.Treeview(frame, columns=columns, show='headings', height=15)
        
        for col in columns:
            self.tree_servicios.heading(col, text=col)
            if col == "Descripción":
                self.tree_servicios.column(col, width=300)
            else:
                self.tree_servicios.column(col, width=120)
        
        scrollbar = ttk.Scrollbar(frame, orient=tk.VERTICAL, command=self.tree_servicios.yview)
        self.tree_servicios.configure(yscrollcommand=scrollbar.set)
        
        self.tree_servicios.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
        
        ttk.Button(frame, text="Actualizar Lista", command=self._actualizar_servicios).pack(pady=10)
    
    def _crear_pestana_reservas(self):
        tab = ttk.Frame(self.notebook)
        self.notebook.add(tab, text="📅 Reservas")
        
        # Panel superior - Formulario
        form_frame = ttk.LabelFrame(tab, text="Nueva Reserva", padding=10)
        form_frame.pack(fill=tk.X, pady=(0, 10))
        
        # Fila 1
        row1 = ttk.Frame(form_frame)
        row1.pack(fill=tk.X, pady=5)
        
        ttk.Label(row1, text="ID Reserva:").pack(side=tk.LEFT, padx=(0, 10))
        self.reserva_id_entry = ttk.Entry(row1, width=15)
        self.reserva_id_entry.pack(side=tk.LEFT, padx=(0, 20))
        
        ttk.Label(row1, text="ID Cliente:").pack(side=tk.LEFT, padx=(0, 10))
        self.reserva_cliente_entry = ttk.Entry(row1, width=15)
        self.reserva_cliente_entry.pack(side=tk.LEFT, padx=(0, 20))
        
        ttk.Label(row1, text="ID Servicio:").pack(side=tk.LEFT, padx=(0, 10))
        self.reserva_servicio_combo = ttk.Combobox(row1, width=20)
        self.reserva_servicio_combo.pack(side=tk.LEFT)
        
        # Fila 2
        row2 = ttk.Frame(form_frame)
        row2.pack(fill=tk.X, pady=5)
        
        ttk.Label(row2, text="Fecha (YYYY-MM-DD HH:MM):").pack(side=tk.LEFT, padx=(0, 10))
        self.reserva_fecha_entry = ttk.Entry(row2, width=20)
        self.reserva_fecha_entry.pack(side=tk.LEFT, padx=(0, 20))
        
        ttk.Label(row2, text="Duración (minutos):").pack(side=tk.LEFT, padx=(0, 10))
        self.reserva_duracion_entry = ttk.Entry(row2, width=10)
        self.reserva_duracion_entry.pack(side=tk.LEFT, padx=(0, 20))
        
        ttk.Label(row2, text="Descuento (%):").pack(side=tk.LEFT, padx=(0, 10))
        self.reserva_descuento_entry = ttk.Entry(row2, width=10)
        self.reserva_descuento_entry.pack(side=tk.LEFT)
        
        # Botones de acción
        btn_frame = ttk.Frame(form_frame)
        btn_frame.pack(fill=tk.X, pady=10)
        
        ttk.Button(btn_frame, text="Crear Reserva", command=self._crear_reserva).pack(side=tk.LEFT, padx=5)
        ttk.Button(btn_frame, text="Confirmar Reserva", command=self._confirmar_reserva).pack(side=tk.LEFT, padx=5)
        ttk.Button(btn_frame, text="Cancelar Reserva", command=self._cancelar_reserva).pack(side=tk.LEFT, padx=5)
        ttk.Button(btn_frame, text="Procesar Reserva", command=self._procesar_reserva).pack(side=tk.LEFT, padx=5)
        ttk.Button(btn_frame, text="Completar Reserva", command=self._completar_reserva).pack(side=tk.LEFT, padx=5)
        
        # ID para operaciones
        oper_frame = ttk.Frame(form_frame)
        oper_frame.pack(fill=tk.X, pady=5)
        ttk.Label(oper_frame, text="ID Reserva para operaciones:").pack(side=tk.LEFT, padx=(0, 10))
        self.operacion_id_entry = ttk.Entry(oper_frame, width=15)
        self.operacion_id_entry.pack(side=tk.LEFT)
        
        # Panel inferior - Lista de reservas
        list_frame = ttk.LabelFrame(tab, text="Lista de Reservas", padding=10)
        list_frame.pack(fill=tk.BOTH, expand=True)
        
        columns = ("ID", "Cliente", "Servicio", "Fecha", "Duración", "Estado", "Costo")
        self.tree_reservas = ttk.Treeview(list_frame, columns=columns, show='headings', height=12)
        
        for col in columns:
            self.tree_reservas.heading(col, text=col)
            self.tree_reservas.column(col, width=130)
        
        scrollbar = ttk.Scrollbar(list_frame, orient=tk.VERTICAL, command=self.tree_reservas.yview)
        self.tree_reservas.configure(yscrollcommand=scrollbar.set)
        
        self.tree_reservas.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
        
        # Botón actualizar
        btn_actualizar = ttk.Frame(list_frame)
        btn_actualizar.pack(fill=tk.X, pady=10)
        ttk.Button(btn_actualizar, text="Actualizar Lista", command=self._actualizar_reservas).pack()
        
        # Evento de selección
        self.tree_reservas.bind('<<TreeviewSelect>>', self._on_reserva_seleccionada)
        
        # Actualizar combo de servicios
        self._actualizar_combo_servicios()
    
    def _crear_pestana_reportes(self):
        tab = ttk.Frame(self.notebook)
        self.notebook.add(tab, text="📊 Reportes")
        
        # Reporte de ingresos
        frame1 = ttk.LabelFrame(tab, text="Reporte de Ingresos", padding=10)
        frame1.pack(fill=tk.X, pady=(0, 10))
        
        self.reporte_text = scrolledtext.ScrolledText(frame1, height=10, width=80, font=('Courier', 10))
        self.reporte_text.pack(fill=tk.BOTH, expand=True)
        
        ttk.Button(frame1, text="Generar Reporte", command=self._generar_reporte).pack(pady=10)
        
        # Logs
        frame2 = ttk.LabelFrame(tab, text="Registro de Eventos (Últimos eventos)", padding=10)
        frame2.pack(fill=tk.BOTH, expand=True)
        
        self.logs_text = scrolledtext.ScrolledText(frame2, height=10, font=('Courier', 9))
        self.logs_text.pack(fill=tk.BOTH, expand=True)
        
        ttk.Button(frame2, text="Actualizar Logs", command=self._actualizar_logs).pack(pady=10)
    
    def _registrar_cliente(self):
        id_cliente = self.entries_clientes["id_cliente"].get()
        nombre = self.entries_clientes["nombre"].get()
        email = self.entries_clientes["email"].get()
        telefono = self.entries_clientes["telefono"].get()
        direccion = self.entries_clientes["direccion"].get()
        vip = self.vip_var.get()
        
        if not all([id_cliente, nombre, email, telefono]):
            messagebox.showwarning("Campos incompletos", "Complete los campos obligatorios")
            return
        
        cliente = self.sistema.registrar_cliente(id_cliente, nombre, email, telefono, direccion, vip)
        if cliente:
            messagebox.showinfo("Éxito", f"Cliente {nombre} registrado exitosamente")
            for entry in self.entries_clientes.values():
                entry.delete(0, tk.END)
            self.vip_var.set(False)
            self._actualizar_clientes()
    
    def _eliminar_cliente(self):
        """Eliminar cliente por ID ingresado manualmente"""
        id_cliente = self.eliminar_id_entry.get().strip()
        
        if not id_cliente:
            messagebox.showwarning("ID requerido", "Ingrese el ID del cliente a eliminar")
            return
        
        # Buscar cliente para mostrar nombre
        cliente = self.sistema.buscar_cliente(id_cliente)
        if not cliente:
            messagebox.showwarning("No encontrado", f"No existe cliente con ID {id_cliente}")
            return
        
        if messagebox.askyesno("Confirmar", f"¿Está seguro de eliminar al cliente:\n{cliente.get_nombre()} (ID: {id_cliente})?"):
            if self.sistema.eliminar_cliente(id_cliente):
                self.eliminar_id_entry.delete(0, tk.END)
                self._actualizar_clientes()
                self._actualizar_reservas()
    
    def _eliminar_cliente_seleccionado(self):
        """Eliminar cliente seleccionado en el treeview"""
        seleccion = self.tree_clientes.selection()
        if not seleccion:
            messagebox.showwarning("Selección requerida", "Seleccione un cliente de la lista")
            return
        
        id_cliente = self.tree_clientes.item(seleccion[0])['values'][0]
        nombre = self.tree_clientes.item(seleccion[0])['values'][1]
        
        if messagebox.askyesno("Confirmar", f"¿Está seguro de eliminar al cliente:\n{nombre} (ID: {id_cliente})?"):
            if self.sistema.eliminar_cliente(id_cliente):
                self.eliminar_id_entry.delete(0, tk.END)
                self._actualizar_clientes()
                self._actualizar_reservas()
    
    def _on_cliente_seleccionado(self, event):
        """Evento cuando se selecciona un cliente en la lista"""
        seleccion = self.tree_clientes.selection()
        if seleccion:
            id_cliente = self.tree_clientes.item(seleccion[0])['values'][0]
            self.eliminar_id_entry.delete(0, tk.END)
            self.eliminar_id_entry.insert(0, id_cliente)
    
    def _crear_reserva(self):
        id_reserva = self.reserva_id_entry.get()
        id_cliente = self.reserva_cliente_entry.get()
        seleccion = self.reserva_servicio_combo.get()
        id_servicio = seleccion.split(" - ")[0] if seleccion else ""
        fecha = self.reserva_fecha_entry.get()
        duracion_str = self.reserva_duracion_entry.get()
        descuento_str = self.reserva_descuento_entry.get()
        
        if not all([id_reserva, id_cliente, id_servicio, fecha, duracion_str]):
            messagebox.showwarning("Campos incompletos", "Complete todos los campos")
            return
        
        try:
            duracion = int(duracion_str)
            parametros = {}
            if descuento_str:
                parametros['descuento'] = float(descuento_str)
            
            reserva = self.sistema.crear_reserva(id_reserva, id_cliente, id_servicio, fecha, duracion, parametros)
            if reserva:
                messagebox.showinfo("Éxito", f"Reserva {id_reserva} creada")
                self.reserva_id_entry.delete(0, tk.END)
                self.reserva_cliente_entry.delete(0, tk.END)
                self.reserva_fecha_entry.delete(0, tk.END)
                self.reserva_duracion_entry.delete(0, tk.END)
                self.reserva_descuento_entry.delete(0, tk.END)
                self._actualizar_reservas()
        except ValueError:
            messagebox.showerror("Error", "Duración debe ser un número válido")
    
    def _confirmar_reserva(self):
        id_reserva = self.operacion_id_entry.get()
        if not id_reserva:
            messagebox.showwarning("ID requerido", "Ingrese el ID de la reserva")
            return
        
        if self.sistema.confirmar_reserva(id_reserva):
            messagebox.showinfo("Éxito", f"Reserva {id_reserva} confirmada")
            self.operacion_id_entry.delete(0, tk.END)
            self._actualizar_reservas()
            self._generar_reporte()
    
    def _cancelar_reserva(self):
        id_reserva = self.operacion_id_entry.get()
        if not id_reserva:
            messagebox.showwarning("ID requerido", "Ingrese el ID de la reserva")
            return
        
        motivo = simpledialog.askstring("Motivo", "Ingrese el motivo de cancelación:")
        if self.sistema.cancelar_reserva(id_reserva, motivo):
            messagebox.showinfo("Éxito", f"Reserva {id_reserva} cancelada")
            self.operacion_id_entry.delete(0, tk.END)
            self._actualizar_reservas()
            self._generar_reporte()
    
    def _procesar_reserva(self):
        id_reserva = self.operacion_id_entry.get()
        if not id_reserva:
            messagebox.showwarning("ID requerido", "Ingrese el ID de la reserva")
            return
        
        if self.sistema.procesar_reserva(id_reserva):
            messagebox.showinfo("Éxito", f"Reserva {id_reserva} en proceso")
            self.operacion_id_entry.delete(0, tk.END)
            self._actualizar_reservas()
    
    def _completar_reserva(self):
        id_reserva = self.operacion_id_entry.get()
        if not id_reserva:
            messagebox.showwarning("ID requerido", "Ingrese el ID de la reserva")
            return
        
        if self.sistema.completar_reserva(id_reserva):
            messagebox.showinfo("Éxito", f"Reserva {id_reserva} completada")
            self.operacion_id_entry.delete(0, tk.END)
            self._actualizar_reservas()
            self._generar_reporte()
    
    def _on_reserva_seleccionada(self, event):
        seleccion = self.tree_reservas.selection()
        if seleccion:
            id_reserva = self.tree_reservas.item(seleccion[0])['values'][0]
            self.operacion_id_entry.delete(0, tk.END)
            self.operacion_id_entry.insert(0, id_reserva)
    
    def _actualizar_clientes(self):
        for item in self.tree_clientes.get_children():
            self.tree_clientes.delete(item)
        
        for cliente in self.sistema.listar_clientes():
            self.tree_clientes.insert('', tk.END, values=(
                cliente.get_id(),
                cliente.get_nombre(),
                cliente.get_email(),
                cliente.get_telefono(),
                "Sí" if cliente.is_vip() else "No",
                f"${cliente.get_total_gastado():,.2f}"
            ))
    
    def _actualizar_servicios(self):
        for item in self.tree_servicios.get_children():
            self.tree_servicios.delete(item)
        
        for servicio in self.sistema.listar_servicios_disponibles():
            self.tree_servicios.insert('', tk.END, values=(
                servicio.get_id(),
                servicio.get_nombre(),
                f"${servicio.get_precio_base():,.2f}",
                f"{servicio.get_duracion_base()} min",
                servicio.describir()
            ))
    
    def _actualizar_reservas(self):
        for item in self.tree_reservas.get_children():
            self.tree_reservas.delete(item)
        
        for reserva in self.sistema.listar_reservas():
            self.tree_reservas.insert('', tk.END, values=(
                reserva.get_id(),
                reserva.get_cliente().get_nombre(),
                reserva.get_servicio().get_nombre(),
                reserva.get_fecha().strftime("%Y-%m-%d %H:%M"),
                f"{reserva.get_duracion()} min",
                reserva.get_estado().value,
                f"${reserva.get_costo():,.2f}"
            ))
    
    def _actualizar_combo_servicios(self):
        servicios = [f"{s.get_id()} - {s.get_nombre()}" for s in self.sistema.listar_servicios_disponibles()]
        self.reserva_servicio_combo['values'] = servicios
    
    def _actualizar_listas(self):
        self._actualizar_clientes()
        self._actualizar_servicios()
        self._actualizar_reservas()
        self._actualizar_combo_servicios()
    
    def _generar_reporte(self):
        ingresos = self.sistema.generar_reporte_ingresos()
        
        reporte = "="*60 + "\n"
        reporte += "REPORTE DE INGRESOS - SOFTWARE FJ\n"
        reporte += "="*60 + "\n\n"
        reporte += f"Total Ingresos: ${ingresos['total_ingresos']:,.2f}\n"
        reporte += f"Total Reservas Completadas: {ingresos['total_reservas_completadas']}\n\n"
        reporte += "Ingresos por Servicio:\n"
        reporte += "-"*40 + "\n"
        
        if ingresos['ingresos_por_servicio']:
            for servicio, monto in ingresos['ingresos_por_servicio'].items():
                reporte += f"  {servicio}: ${monto:,.2f}\n"
        else:
            reporte += "  No hay ingresos registrados\n"
        
        reporte += "\n" + "="*60 + "\n"
        reporte += f"Total Clientes: {len(self.sistema.listar_clientes())}\n"
        reporte += f"Total Reservas: {len(self.sistema.listar_reservas())}\n"
        
        # Contar reservas por estado
        estados = {}
        for r in self.sistema.listar_reservas():
            estado = r.get_estado().value
            estados[estado] = estados.get(estado, 0) + 1
        
        reporte += "\nReservas por estado:\n"
        for estado, cantidad in estados.items():
            reporte += f"  {estado}: {cantidad}\n"
        
        self.reporte_text.delete(1.0, tk.END)
        self.reporte_text.insert(1.0, reporte)
    
    def _actualizar_logs(self):
        try:
            with open("sistema_fj.log", 'r', encoding='utf-8') as f:
                lines = f.readlines()
                ultimas = lines[-30:] if len(lines) > 30 else lines
                self.logs_text.delete(1.0, tk.END)
                self.logs_text.insert(1.0, ''.join(ultimas))
        except:
            self.logs_text.delete(1.0, tk.END)
            self.logs_text.insert(1.0, "No se pudieron cargar los logs")


def main():
    root = tk.Tk()
    app = AplicacionFJ(root)
    root.mainloop()


if __name__ == "__main__":
    main()
