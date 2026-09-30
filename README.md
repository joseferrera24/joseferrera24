import asyncio
import random
from playwright.async_api import async_playwright

# Lista de configuración de tus perfiles (puedes cargar esto desde un archivo JSON o base de datos)
PERFILES_CONFIG = [
    {
        "nombre": "perfil_youtube_01",
        "proxy": "http://usuario:password@ip_proxy_1:puerto",  # Cambia por tu proxy o deja None
        "url": "https://www.youtube.com"
    },
    {
        "nombre": "perfil_youtube_02",
        "proxy": "http://usuario:password@ip_proxy_2:puerto",  # Cambia por tu proxy o deja None
        "url": "https://www.youtube.com"
    },
    {
        "nombre": "perfil_youtube_03",
        "proxy": None,  # Ejemplo sin proxy (conexión directa)
        "url": "https://www.youtube.com"
    }
]

async def ejecutar_perfil_automatizado(config):
    nombre_perfil = config["nombre"]
    proxy_url = config["proxy"]
    url_destino = config["url"]

    async with async_playwright() as p:
        # 1. Directorio de perfil aislado para guardar cookies y sesión de forma independiente
        user_data_dir = f"./perfiles_usuario/{nombre_perfil}"
        
        # 2. Configurar el proxy si existe
        proxy_config = {"server": proxy_url} if proxy_url else None

        # 3. Lanzar el navegador persistente
        browser_context = await p.chromium.launch_persistent_context(
            user_data_dir=user_data_dir,
            headless=False,  # Manténlo en False para ver las ventanas abrirse
            proxy=proxy_config,
            args=[
                "--disable-blink-features=AutomationControlled",
                "--start-maximized" # Opcional: para que se abran maximizadas
            ]
        )

        # Si no hay páginas abiertas, creamos una nueva
        page = browser_context.pages[0] if browser_context.pages else await browser_context.new_page()
        
        try:
            print(f"[{nombre_perfil}] Abriendo navegador y conectando a {url_destino}...")
            await page.goto(url_destino)
            
            # Simular ciclos de actividad y descanso repetitivos
            ciclos = 3
            for i in range(ciclos):
                print(f"[{nombre_perfil}] Ciclo {i+1}: Realizando actividad...")
                
                # Simular tiempo de reproducción o interacción aleatoria
                await page.wait_for_timeout(random.randint(6000, 12000))
                
                # Ciclo de descanso aleatorio para simular comportamiento humano
                tiempo_descanso = random.randint(4000, 8000)
                print(f"[{nombre_perfil}] Entrando en descanso por {tiempo_descanso / 1000} segundos...")
                await asyncio.sleep(tiempo_descanso / 1000)
                
        except Exception as e:
            print(f"[{nombre_perfil}] Error detectado: {e}")
        finally:
            await browser_context.close()
            print(f"[{nombre_perfil}] Ventana cerrada y recursos liberados.")

async def main():
    # Creamos una tarea en paralelo para cada perfil configurado en la lista
    tareas = [ejecutar_perfil_automatizado(perfil) for perfil in PERFILES_CONFIG]
    
    # Ejecutamos todas las ventanas al mismo tiempo
    await asyncio.gather(*tareas)

if __name__ == "__main__":
    asyncio.run(main())
