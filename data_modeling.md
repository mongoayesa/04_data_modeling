# Data Model Patterns

En principio MongoDB habla de dos modelos de datos:

## Modelo de datos denormalizado (documentos embebidos)

El modelo de datos denormalizado es el óptimo e ideal para MongoDB y al que se debe
recurrir siempre que sea posible

Ej:

Colección productos

{
    producto: "Zapatillas ZXV",
    marca: "Nike",
    distribuidores: [
        {nombre: "ServiZapas", contacto: "...", ...},
        {nombre: "Distribuciones Pérez", contacto: "...", ...}
    ]
}

## Modelo de datos normalizado (referencias entre colecciones)

Modelo a evitar siempre que se pueda

Ej:

Colección productos

{
    producto: "Zapatillas ZXV",
    marca: "Nike",
    distribuidores: [1, 2, ...]
}

Colección distribuidores

{_id: 1, nombre: "ServiZapas", contacto: "...", ...},
{_id: 2, nombre: "Distribuciones Pérez", contacto: "...", ...}
...

# ¿Qué modelo escoger en cada caso?

## Patrón de relaciones One-to-one

- Siempre se usará el modelo denormalizado
- Con una sola consulta se consiguen todos los datos

Ej:

Colección usuarios

{
    _id: 3,
    nombre: "John Doe",
    dirección: {  // Relación one-to-one 1 usuario 1 dirección
        calle: "Gran Vía, 80",
        cp: "28001",
        localidad: "Madrid"
    }
}

## Patrón de relaciones One-to-few

Usaremos también el modelo denormalizado, siempre que:
- El lado one sea el que normalmente reciba más consultas
- No serán frecuentes las escrituras en el lado few


Ej:

Colección productos

{
    producto: "Zapatillas ZXV",
    marca: "Nike",
    imagenes: [
        {url: "https://dominio/ckjsdjhcs.jpg", textoAlt: "..."},
        {url: "https://dominio/hgfhfhchc.jpg", textoAlt: "..."},
        {url: "https://dominio/dtrdtdtdz.jpg", textoAlt: "..."},
        ...
    ]
}

## Patrón de relaciones One-to-many

Hay que estudiar cada caso y en función de la posible evolución del modelo
de datos seleccionar denormalizado o normalizado.

- Si se preve que la parte many escalará hacia muchos valores sería conveniente
el modelo normalizado para evitar superar el límite de 16MB de los docs MongoDB

## Patrón de relaciones One-to-skillions

Se utilizará el modelo normalizado

Colección productos

{
    _id: "csjdgsj1123"
    producto: "Zapatillas ZXV",
    marca: "Nike",
    opiniones: [ "13212gjhgh", "sdgajshg32", "axgs1x2431"]
}

Colección opiniones

{ _id: "13212gjhgh", id_producto: csjdgsj1123, texto: 'buen producto...', estrellas: 3, ...}
{ _id: "sdgajshg32", id_producto: csjdgsj1123, texto: 'buen producto...', estrellas: 3, ...}
{ _id: "axgs1x2431", id_producto: csjdgsj1123, texto: 'buen producto...', estrellas: 3, ...}
...

## Patrón de relaciones Many-to-Many

Dependerá de cada caso, pero como aproximación podemos tener en cuenta lo siguiente:

- Modelo denormalizado si se cumple:
    - Mayor número de consultas se produce en el lado con mayor número de registros
    - Podemos tener redundancia de datos

Ej:

Colección productos

{
    _id: "cjshdgcjs56",
    producto: "Nike FTV",
    marca: "Nike",
    tiendas: [
        {nombre: "Alcorcón Store", calle: "...", contacto: "..."},
        {nombre: "Las Rozas Store", calle: "...", contacto: "..."},
    ]
}

{
    _id: "hfsjfhsk45w3",
    producto: "Adidas Tokyo",
    marca: "Adidas",
    tiendas: [
        {nombre: "Alcorcón Store", calle: "...", contacto: "..."},
    ]
}

- Modelo denormalizado siempre que:
    - Las consultas se produzcan con la misma frecuencia en un lado que en otro
    - La existencia de redundancia pueda ocasionar problemas

Ej:

Colección productos

{
    _id: "cjshdgcjs56",
    producto: "Nike FTV",
    marca: "Nike",
    tiendas: ["vkhkdsh76756","csgsud454333"]
}

{
    _id: "hfsjfhsk45w3",
    producto: "Adidas Tokyo",
    marca: "Adidas",
    tiendas: ["csgsud454333"]
}

Colección de tiendas

{
    _id: "vkhkdsh76756", 
    nombre: "Alcorcón Store", 
    calle: "...", 
    contacto: "...", 
    productos: ["cjshdgcjs56"]
},
{
    _id: "csgsud454333", 
    nombre: "Las Rozas Store", 
    calle: "...", 
    contacto: "...", 
    productos: ["cjshdgcjs56", "hfsjfhsk45w3"]
},
