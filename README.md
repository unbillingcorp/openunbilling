# Open Unbilling

DISCLAIMER

Es una iniciativa en la que pretendemos ofrecer a la comunidad de un estándar de transferencia de datos EDI, para enviar comprobantes electrónicos hacia el Ministerio de Hacienda de Costa Rica, a través de la plataforma Unbilling, la cual se hará cargo de la gestión y almacenamiento de las facturas, tiquetes, notas de crédito y de débito. 

Para ello disponemos de un metodo simple, sencillo y abierto soportado bajo OneDrive, para recibir un listado de líneas con el encabezado, detalle y resumen de los comprobantes electrónicos en un formato plano, de fácil entendimiento y abstracción para la integración con cualquier aplicativo POS o sistema ERP, que genere ventas para ser declaradas posteriormente con la plataforma Unbilling, no siendo limitativo para éste el estándar aquí dispuesto.

De parte de los providers, desarrolladores de software o ISV, entregamos una manera documentada a través de este repositorio de GitHub para que construyan las interfaces requeridas, las cuales pueden trabajar tanto en línea como por lotes para el envío de los comprobantes electrónicos a la plataforma Unbilling, de manera tal de no tener que lidiar con los estándares y procesos internos con el Ministerio de Hacienda, asi como también evitar la necesidad de realizar desarrollos mucho más elaborados para lidiar con la gestión de factura electrónica en Costa Rica, contando para ello del soporte backend de Unbilling.

# Esquema propuesto ( Archivo .JSON )

El esquema propuesto, para el envío de comprobantes al Ministerio de Hacienda por medio de la plataforma Unbilling, es que éste se haga mediante la colocación de un archivo en formato JavaScript Object Notation ( .JSON ), con la información a procesar y enviar. La cantidad de datos no tiene límite, por lo tanto, se pueden recibir de 1 a N registros, sin inconveniente.

La descripción detallada de los datos que se esperan en los registros del archivo, es la siguiente:

1. Número consecutivo ( Tipo del dato: Entero )
Es un número de Consecutivo interno que maneje el emisor. Por ejemplo, un consecutivo de número de venta, un número de orden, un consecutivo interno de ERP, etc.

2. Receptor ( Tipo del dato: Objeto con propiedades )
Datos de la persona física o jurídica, que va a recibir el comprobante, cuando es factura. En el caso que sea un Tiquete puede venir con un valor de nulo. 
Los datos enviados, que conforman la información básica del receptor, es la siguiente: 

- Nombre del receptor.
- Tipo de identificación del receptor ( basado en los códigos de tipo de identificación del archivo "ANEXOS Y ESTRUCTURAS_V4.3.pdf" del Ministerio de Hacienda. Nota 4 ).
- Número de identificación del receptor ( este debe cumplir el formato que se indica en el archivo "ANEXOS Y ESTRUCTURAS_V4.3.pdf" del Ministerio de Hacienda. Nota 4.1 ).
- Correo electrónico del receptor.
- Código del país al que corresponde el teléfono del receptor ( este es un número entero ).
- Teléfono del receptor ( este es un número entero ).

El formato esperado es: 
"Receptor": 
{
	"Nombre": "....",
	"TipoIdentificacion": "",
	"Identificacion": "",
	"Correo": "",
	"CodigoPaisTelefono": XXX,
	"Telefono": XXXXXXXX
}

Ejemplo de datos para esta columna es: 
"Receptor": 
{
	"Nombre": "juan",
	"TipoIdentificacion": "01",
	"Identificacion": "303330444",
	"Correo": "juan@correo.com",
	"CodigoPaisTelefono": 506,
	"Telefono": 22334455
}

3. Condición de Venta ( Tipo del dato: Cadena de 2 caracteres )
Corresponde al Código de la condición de venta, que ha sido definido por el Ministerio de Hacienda, según los siguiente valores: 
"01" - Contado 
"02" - Crédito
"03" - Consignación
"04" - Apartado
"05" - Arrendamiento con opción de compra
"06" - Arrendamiento en función financiera
"07" - Cobro a favor de un tercero
"08" - Servicios prestados al Estado a crédito
"09" - Pago del servicios prestado al Estado
"99" - Otros (se debe indicar la condición de la venta)

4. Medio de pago  ( Tipo del dato: Cadena de 2 caracteres )
Corresponde al Código del medio de pago, que ha sido definido por el Ministerio de Hacienda, según los siguiente valores: 
"01" - Efectivo
"02" - Tarjeta
"03" - Cheque
"04" - Transferencia – depósito bancario
"05" - Recaudado por terceros
"99" - Otros (se debe indicar el medio de pago)

5. Tipo de comprobante ( Tipo del dato: Cadena de 2 caracteres )
Indica el tipo de comprobante que se debe emitir, para la línea correspondiente.
Los valores esperados son: 
"FA": factura
"TI": tiquete

6. Moneda del comprobante ( Tipo del dato: Objeto con propiedades )
Datos de la moneda en la cual se está realizando la transacción y que se va a registrar en el comprobante respectivo. 
Los datos que se esperan, son los siguientes: 
- Código de la moneda ( Código de 3 caracteres de la moneda. Ver https://www.currency-iso.org/en/home/tables/table-a1.html )
- Tipo de cambio ( 1 para colón o el Tipo de Cambio que tiene la moneda del comprobante en ese momento )

Ejemplo del formato: 
"Moneda": 
{
	"Codigo": "",
	"TipoCambio": XXXX.XX
}

Ejemplo de datos esperados: 
"Moneda": 
{
	"Codigo": "USD",
	"TipoCambio": 601.55
}

7. Lista de Productos/Servicios  ( Tipo del dato: Objeto con propiedades )
En esta sección del archivo se define el listado de los productos o servicios, que conforman el comprobante y que van a ser enviados al Ministerio de Hacienda.
El detalle de los datos de cada artículo en el listado, es el siguiente:

- Cantidad: Cantidad vendida del artículo o del servicio prestado. ( Número decimal con "." como separador de decimales y "," como separador de miles ).
- Detalle: Descripción del artículo o servicio prestado ( Cadena de caracteres )
- Precio unitario del producto/servicio. ( Número decimal con "." como separador de decimales y "," como separador de miles )
- Unidad de medida ( Basado en el archivo "ANEXOS Y ESTRUCTURAS_V4.3.pdf" del Ministerio de Hacienda. Nota 15 )
- Código: Se refiere al código Cabys, definido por el Ministerio de Hacienda para los Bienes y Servicios.
- Descuento: Estructura que define el detalle del descuento aplicado al producto/servicio.
	- Monto del descuento ( Número decimal con "." como separador de decimales y "," como separador de miles )
	- Descripción del descuento ( Cadena de caracteres )
- Código Comercial del producto/servicio
	- Código: Descripción del código del producto/servicio. ( Cadena de caracteres )
	- Tipo de código que ha sido definido por el Ministerio de Hacienda, según los siguiente valores: ( Cadena de 2 caracteres )
	  "01" - Código del producto del vendedor
	  "02" - Código del producto del comprador
	  "03" - Código del producto asignado por la industria
	  "04" - Código uso interno
	  "99" - Otros
- Impuesto: Es un listado con el Detalle del(los) impuesto(s) que aplica(n) al artículo o servicio. 
Los valores que se esperan, por cada impuesto, son los siguientes:
	- Código de impuesto: que ha sido definido por el Ministerio de Hacienda, según los siguiente valores: ( Cadena de 2 caracteres ) 
		"01" - Impuesto al Valor Agregado
		"02" - Impuesto Selectivo de Consumo
		"03" - Impuesto Único a los Combustibles
		"04" - Impuesto específico de Bebidas Alcohólicas
		"05" - Impuesto Específico sobre las bebidas envasadas sin contenido alcohólico y jabones de tocador
		"06" - Impuesto a los Productos de Tabaco
		"07" - IVA (cálculo especial)
		"08" - IVA Régimen de Bienes Usados (Factor)
		"12" - Impuesto Específico al Cemento
		"99" - Otros
	- Código de Tarifa: que ha sido definido por el Ministerio de Hacienda, según los siguiente valores: ( Cadena de 2 caracteres . Basado en el archivo "ANEXOS Y ESTRUCTURAS_V4.3.pdf" del Ministerio de Hacienda. Nota 8.1 )
		"01" - Tarifa 0% (Exento)
		"02" - Tarifa reducida 1%
		"03" - Tarifa reducida 2%
		"04" - Tarifa reducida 4%
		"05" - Transitorio 0%
		"06" - Transitorio 4%
		"07" - Transitorio 8%
		"08" - Tarifa general 13%
	- Tarifa: Se toma del valor del porcentaje de la tabla que define el Código de Tarifa. ( Número decimal con "." como separador de decimales y "," como separador de miles )

Ejemplo del formato: 
"Productos": [{
                "Cantidad": XX.XX,
                "Detalle": "",
                "PrecioUnitario": XXX.XX,
                "UnidadMedida": "",
                "CodigoCabys": "CODIGOCABYS",
                "Descuentos": [{
                        "Monto": XX.XX,
                        "Descripcion": ""
                    }
                ],
                "CodigoComercial": {
                    "Codigo": "CODIGODEFINIDO",
                    "Tipo": ""
                },
                "Impuestos": [{
                        "Codigo": "",
                        "CodigoTarifa": "",
                        "Tarifa": XX.XX
                    }
                ]
            }
        ]

Ejemplo de datos esperados: 
 "Productos": [{
                "Cantidad": 2.00,
                "Detalle": "Este es un producto de prueba",
                "PrecioUnitario": 100.00,
                "UnidadMedida": "Unid",
                "CodigoCabys": "2820203010100",
                "Descuentos": [{
                        "Monto": 20.00,
                        "Descripcion": "Descuento promocional"
                    }
                ],
                "CodigoComercial": {
                    "Codigo": "ART 2001-15",
                    "Tipo": "01"
                },
                "Impuestos": [{
                        "Codigo": "01",
                        "CodigoTarifa": "08",
                        "Tarifa": 13.00
                    }
                ]
            }
        ]



Ejemplo de contenido esperado en el archivo JSON.

Factura con un artículo.

{
        "Consecutivo": 10,
        "Receptor": {
            "Nombre": "juan",
            "TipoIdentificacion": "01",
            "Identificacion": "303330444",
            "Correo": "juan@correo.com",
            "CodigoPaisTelefono": 506,
            "Telefono": 22334455
        },
        "CondicionVenta": "01",
        "MedioPago": "02",
        "TipoComprobante": "FA",
        "Moneda": {
            "Codigo": "CRC",
            "TipoCambio": 1
        },
        "Productos": [{
                "Cantidad": 2.00,
                "Detalle": "Este es un producto de prueba",
                "PrecioUnitario": 100.00,
                "UnidadMedida": "Unid",
                "CodigoCabys": "2820203010100",
                "Descuentos": [{
                        "Monto": 20.00,
                        "Descripcion": "Descuento promocional"
                    }
                ],
                "CodigoComercial": {
                    "Codigo": "ART 2001-15",
                    "Tipo": "01"
                },
                "Impuestos": [{
                        "Codigo": "01",
                        "CodigoTarifa": "08",
                        "Tarifa": 13.00
                    }
                ]
            }
        ]
}

Tiquete con dos artículos.

{
        "Consecutivo": 20,
        "Receptor": null,
        "CondicionVenta": "02",
        "MedioPago": "01",
        "TipoComprobante": "TI",
        "Moneda": {
            "Codigo": "USD",
            "TipoCambio": 602.55
        },
        "Productos": [{
                "Cantidad": 2.00,
                "Detalle": "Este es un producto de prueba",
                "PrecioUnitario": 100.00,
                "UnidadMedida": "Unid",
                "CodigoCabys": "2820203010100",
                "Descuentos": [{
                        "Monto": 20.00,
                        "Descripcion": "Descuento promocional"
                    }
                ],
                "CodigoComercial": {
                    "Codigo": "ART 2001-15",
                    "Tipo": "01"
                },
                "Impuestos": [{
                        "Codigo": "01",
                        "CodigoTarifa": "08",
                        "Tarifa": 13.00
                    }
                ]
            }, {
                "Cantidad": 1.00,
                "Detalle": "Este es otro producto de prueba",
                "PrecioUnitario": 100.00,
                "UnidadMedida": "Unid",
                "CodigoCabys": "2820203010199",
                "Descuentos": [{
                        "Monto": 10.00,
                        "Descripcion": "Descuento promocional"
                    }
                ],
                "CodigoComercial": {
                    "Codigo": "ART 2001-99",
                    "Tipo": "01"
                },
                "Impuestos": [{
                        "Codigo": "01",
                        "CodigoTarifa": "08",
                        "Tarifa": 13.00
                    }
                ]
            }
        ]

# Esquema propuesto ( Archivo .CSV )

El esquema propuesto, para el envío de comprobantes al Ministerio de Hacienda por medio de la plataforma Unbilling, es que éste se haga mediante la colocación de un archivo separado por comas ( .CSV ), con los datos a procesar y enviar.

El formato del archivo que se espera recibir, está compuesto por 7 columnas. Una breve descripción del contenido de las columnas, se da a continuación:

- Columna 1: Número de consecutivo
- Columna 2: Datos del Receptor
- Columna 3: Condición de la Venta
- Columna 4: Medio de Pago
- Columna 5: Tipo de Comprobante a emitir
- Columna 6: Datos de la moneda en que se emitirá el comprobante
- Columna 7: Listado de productos o servicios que se enviarán al Ministerio de Hacienda

En este archivo, el emisor podrá enviar los datos de los comprobantes que desea que sean transmitidos al Ministerio de Hacienda, a través de Unbilling. La cantidad de líneas de datos no tiene límite, por lo tanto, se pueden recibir de 1 a N líneas a procesar, sin inconveniente.

La descripción detallada de los datos que se esperan en las líneas del archivo, en cada columna es la siguiente:

1. Número consecutivo ( Tipo del dato: Entero )
Es un número de Consecutivo interno que maneje el emisor. Por ejemplo, un consecutivo de número de venta, un número de orden, un consecutivo interno de ERP, etc.

2. Receptor ( Tipo del dato: Cadena de caracteres )
Datos de la persona física o jurídica, que va a recibir el comprobante, cuando es factura. Los datos enviados, que conforman la información básica del receptor, deben venir separados por el caracter "|".
En esta columna se espera:
- Nombre del receptor.
- Tipo de identificación del receptor ( basado en los códigos de tipo de identificación del archivo "ANEXOS Y ESTRUCTURAS_V4.3.pdf" del Ministerio de Hacienda. Nota 4 ).
- Número de identificación del receptor ( este debe cumplir el formato que se indica en el archivo "ANEXOS Y ESTRUCTURAS_V4.3.pdf" del Ministerio de Hacienda. Nota 4.1 ).
- Correo electrónico del receptor.
- Código del país al que corresponde el teléfono del receptor ( este es un dato numérico ).
- Teléfono del receptor ( este es un dato numérico ).

El formato esperado es: 
Nombre|Tipo identificacion|Número identificación|Correo|Codigo país|Teléfono

Ejemplo de datos para esta columna es: 
Juan Pérez|"01"|"303330444"|"juan@gmail.com"|506|88990000

3. Condición de Venta ( Tipo del dato: Cadena de 2 caracteres )
Corresponde al Código de la condición de venta, que ha sido definido por el Ministerio de Hacienda, según los siguiente valores: 
"01" - Contado 
"02" - Crédito
"03" - Consignación
"04" - Apartado
"05" - Arrendamiento con opción de compra
"06" - Arrendamiento en función financiera
"07" - Cobro a favor de un tercero
"08" - Servicios prestados al Estado a crédito
"09" - Pago del servicios prestado al Estado
"99" - Otros (se debe indicar la condición de la venta)

4. Medio de pago  ( Tipo del dato: Cadena de 2 caracteres )
Corresponde al Código del medio de pago, que ha sido definido por el Ministerio de Hacienda, según los siguiente valores: 
"01" - Efectivo
"02" - Tarjeta
"03" - Cheque
"04" - Transferencia – depósito bancario
"05" - Recaudado por terceros
"99" - Otros (se debe indicar el medio de pago)

5. Tipo de comprobante ( Tipo del dato: Cadena de 2 caracteres )
Indica el tipo de comprobante que se debe emitir, para la línea correspondiente.
Los valores esperados son: 
"FA": factura
"TI": tiquete

6. Moneda del comprobante ( Tipo del dato: Cadena de caracteres )
Datos de la moneda en la cual se está realizando la transacción y que se va a registrar en el comprobante respectivo. Los datos enviados, deben venir separados por el caracter "|".
Los datos que se esperan, son los siguientes: 
- Código de la moneda ( Código de 3 caracteres de la moneda. Ver https://www.currency-iso.org/en/home/tables/table-a1.html )
- Tipo de cambio ( 1 para colón o el Tipo de Cambio que tiene la moneda del comprobante en ese momento )

Ejemplo del formato: 
CODIGO|TIPO DE CAMBIO

Ejemplo de datos esperados: 
"USD"|601.50

7. Lista de Productos/Servicios  ( Tipo del dato: Cadena de caracteres )
En esta columna se espera el listado de los productos o servicios, que conforman el comprobante y que van a ser enviados al Ministerio de Hacienda. Los datos de cada artículo deben venir definidos entre llaves "{...}". El detalle del artículo, deben venir separado por el caracter "|", es decir "{...|...|...|...}".

El detalle de los datos de cada artículo en el listado, es el siguiente:

- Cantidad: Cantidad vendida del artículo o del servicio prestado. ( Número decimal con "." como separador de decimales y "," como separador de miles ).
- Detalle: Descripción del artículo o servicio prestado ( Cadena de caracteres )
- Precio unitario del producto/servicio. ( Número decimal con "." como separador de decimales y "," como separador de miles )
- Unidad de medida ( Basado en el archivo "ANEXOS Y ESTRUCTURAS_V4.3.pdf" del Ministerio de Hacienda. Nota 15 )
- Código: Se refiere al código Cabys, definido por el Ministerio de Hacienda para los Bienes y Servicios.
- Descuento: Listado con el Detalle del(los) descuento(s) que aplica(n) al artículo o servicio. La estructura para cada descuento, debe estar definido entre "<...>". Los valores esperados, para cada descuento, son los siguientes:
	- Monto del descuento ( Número decimal con "." como separador de decimales y "," como separador de miles )
	- Descripción del descuento ( Cadena de caracteres )
- Código Comercial del producto/servicio
	- Código: Descripción del código del producto/servicio. ( Cadena de caracteres )
	- Tipo de código que ha sido definido por el Ministerio de Hacienda, según los siguiente valores: ( Cadena de 2 caracteres )
	  "01" - Código del producto del vendedor
	  "02" - Código del producto del comprador
	  "03" - Código del producto asignado por la industria
	  "04" - Código uso interno
	  "99" - Otros
- Impuesto: Es un listado con el Detalle del(los) impuesto(s) que aplica(n) al artículo o servicio. La estructura para cada impuesto, debe estar definido entre "<...>".
Los valores que se esperan, por cada impuesto, son los siguientes:
	- Código de impuesto: que ha sido definido por el Ministerio de Hacienda, según los siguiente valores: ( Cadena de 2 caracteres ) 
		"01" - Impuesto al Valor Agregado
		"02" - Impuesto Selectivo de Consumo
		"03" - Impuesto Único a los Combustibles
		"04" - Impuesto específico de Bebidas Alcohólicas
		"05" - Impuesto Específico sobre las bebidas envasadas sin contenido alcohólico y jabones de tocador
		"06" - Impuesto a los Productos de Tabaco
		"07" - IVA (cálculo especial)
		"08" - IVA Régimen de Bienes Usados (Factor)
		"12" - Impuesto Específico al Cemento
		"99" - Otros
	- Código de Tarifa: que ha sido definido por el Ministerio de Hacienda, según los siguiente valores: ( Cadena de 2 caracteres . Basado en el archivo "ANEXOS Y ESTRUCTURAS_V4.3.pdf" del Ministerio de Hacienda. Nota 8.1 )
		"01" - Tarifa 0% (Exento)
		"02" - Tarifa reducida 1%
		"03" - Tarifa reducida 2%
		"04" - Tarifa reducida 4%
		"05" - Transitorio 0%
		"06" - Transitorio 4%
		"07" - Transitorio 8%
		"08" - Tarifa general 13%
	- Tarifa: Se toma del valor del porcentaje de la tabla que define el Código de Tarifa. ( Número decimal con "." como separador de decimales y "," como separador de miles )

Ejemplo de contenido esperado.

Factura con un artículo.

Consecutivo, Receptor, CondicionVenta, MedioPago, TipoComprobante, Moneda, Productos
10, Juan|"01"|"303330444"|"juan@gmail.com"|506|88990000, "01", "02", FA, "CRC"|1, {2.00|Producto de prueba 1|100.00|Unid|2820203010100|<20.00|Descuento promo>|ART 2001-15|"01"|<"01"|"08"|13.00>}

Tiquete con dos artículos.

Consecutivo, Receptor, CondicionVenta, MedioPago, TipoComprobante, Moneda, Productos
20, , "02", "01", TI, "USD"|602.55, {2.00|Producto de prueba 1|100.00|Unid|2820203010100|<20.00|Descuento promo>|ART 2001-15|"01"|<"01"|"08"|13.00>}{1.00|Producto de prueba 2|100.00|Unid|2820203010100|<10.00|Descuento promo>|ART 2001-05|"01"|<"01"|"08"|13.00>}

