import urllib.request
import xml.etree.ElementTree as ET

from bs4 import BeautifulSoup
from datetime import datetime, timezone
from email.utils import format_datetime
from pathlib import Path
from urllib.parse import urljoin

WEB_URL = "https://www.cirsa.com/prensa/"
BASE_URL = "https://www.cirsa.com"
OUTPUT_FILE = Path("cirsa.xml")

MESES = {
    "Ene": 1,
    "Feb": 2,
    "Mar": 3,
    "Abr": 4,
    "May": 5,
    "Jun": 6,
    "Jul": 7,
    "Ago": 8,
    "Sep": 9,
    "Oct": 10,
    "Nov": 11,
    "Dic": 12,
}


def descargar_noticias():
    solicitud = urllib.request.Request(
        WEB_URL,
        headers={
            "User-Agent": (
                "Mozilla/5.0 (Windows NT 10.0; Win64; x64) "
                "AppleWebKit/537.36 Chrome/124 Safari/537.36"
            ),
            "Accept": "text/html",
        },
    )

    with urllib.request.urlopen(solicitud, timeout=60) as respuesta:
        contenido = respuesta.read()

    soup = BeautifulSoup(contenido, "html.parser")

    noticias = []
    enlaces_encontrados = set()

    for tarjeta in soup.select(
        "#contenido-respuesta-prensa a.news"
    ):
        enlace = urljoin(
            BASE_URL,
            tarjeta.get("href", ""),
        )

        titulo_elemento = tarjeta.select_one(
            ".content-text p"
        )

        fecha_elementos = tarjeta.select(
            ".content-date p"
        )

        imagen_elemento = tarjeta.select_one("img")

        if not enlace or not titulo_elemento:
            continue

        titulo = titulo_elemento.get_text(" ", strip=True)
        imagen = ""

        if imagen_elemento:
            imagen = urljoin(
                BASE_URL,
                imagen_elemento.get("src", ""),
            )

        fecha = None

        if len(fecha_elementos) >= 3:
            dia_texto = fecha_elementos[0].get_text(
                " ",
                strip=True,
            )
            mes_texto = fecha_elementos[1].get_text(
                " ",
                strip=True,
            )
            anio_texto = fecha_elementos[2].get_text(
                " ",
                strip=True,
            )

            try:
                mes = MESES.get(mes_texto[:3].title())

                if mes:
                    fecha = datetime(
                        int(anio_texto),
                        mes,
                        int(dia_texto),
                        tzinfo=timezone.utc,
                    )
            except ValueError:
                pass

        if (
            not titulo
            or enlace in enlaces_encontrados
        ):
            continue

        enlaces_encontrados.add(enlace)

        noticias.append(
            {
                "titulo": titulo,
                "enlace": enlace,
                "fecha": fecha,
                "imagen": imagen,
            }
        )

    return noticias


def crear_rss(noticias):
    rss = ET.Element(
        "rss",
        {
            "version": "2.0",
            "xmlns:atom": "http://www.w3.org/2005/Atom",
            "xmlns:media": "http://search.yahoo.com/mrss/",
        },
    )

    canal = ET.SubElement(rss, "channel")

    ET.SubElement(canal, "title").text = (
        "Notas de prensa de CIRSA"
    )
    ET.SubElement(canal, "link").text = WEB_URL
    ET.SubElement(canal, "description").text = (
        "Últimas noticias y comunicaciones corporativas de CIRSA"
    )
    ET.SubElement(canal, "language").text = "es"
    ET.SubElement(canal, "lastBuildDate").text = format_datetime(
        datetime.now(timezone.utc)
    )

    enlace_atom = ET.SubElement(
        canal,
        "{http://www.w3.org/2005/Atom}link",
    )
    enlace_atom.set("href", WEB_URL)
    enlace_atom.set("rel", "self")
    enlace_atom.set("type", "application/rss+xml")

    for noticia in noticias:
        elemento = ET.SubElement(canal, "item")

        ET.SubElement(
            elemento,
            "title",
        ).text = noticia["titulo"]

        ET.SubElement(
            elemento,
            "link",
        ).text = noticia["enlace"]

        ET.SubElement(
            elemento,
            "description",
        ).text = noticia["titulo"]

        ET.SubElement(
            elemento,
            "category",
        ).text = "Notas de prensa"

        identificador = ET.SubElement(elemento, "guid")
        identificador.set("isPermaLink", "true")
        identificador.text = noticia["enlace"]

        if noticia["fecha"]:
            ET.SubElement(
                elemento,
                "pubDate",
            ).text = format_datetime(noticia["fecha"])

        if noticia["imagen"]:
            imagen = ET.SubElement(
                elemento,
                "{http://search.yahoo.com/mrss/}content",
            )
            imagen.set("url", noticia["imagen"])
            imagen.set("medium", "image")

    ET.indent(rss, space="  ")

    ET.ElementTree(rss).write(
        OUTPUT_FILE,
        encoding="utf-8",
        xml_declaration=True,
    )


def main():
    noticias = descargar_noticias()

    if not noticias:
        raise RuntimeError(
            "No se encontraron notas de prensa de CIRSA"
        )

    crear_rss(noticias)

    print(
        f"RSS creada correctamente con "
        f"{len(noticias)} noticias"
    )


if __name__ == "__main__":
    main()
