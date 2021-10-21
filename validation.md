# Validation en MongoDB
A pesar de que mongo su filosofía es schemaless se puede implementar validación a nivel de colección.

La validación se puede implementar tanto en la creación como a posteriori

## En la creación de colección

Sintaxis
db.createCollection(<colección>, {validator: <$jsonSchema>})

```
use clinica2

db.createCollection("pacientes", {
    validator: {
        $jsonSchema: {
            bsonType: "object",
            required: ["nombre","apellidos","dni"],
            properties: {
                nombre: {
                    bsonType: "string",
                    description: "debe ser strig y es obligatorio"
                },
                apellidos: {
                    bsonType: "string",
                    description: "debe ser strig y es obligatorio"
                },
                dni: {
                    bsonType: "string",
                    description: "debe ser strig y es obligatorio"
                },
                edad: {
                    bsonType: ["number","null"],
                    minimum: 0,
                    maximum: 120,
                    description: "debe ser number entre 0 y 120 ó null" 
                },
                direccion: {
                    bsonType: "object",
                    properties: {
                        calle: {
                            bsonType: "string",
                            description: "debe ser strig"
                        },
                        cp: {
                            bsonType: "string",
                            description: "debe ser strig"
                        },
                        localidad: {
                            bsonType: "string",
                            enum: ["Sevilla","Málaga","Cádiz"],
                            description: "debe ser Sevilla, Málaga o Cádiz"
                        }
                    }
                }
            }
        }
    }
})

db.pacientes.insert({nombre: 'Juan', apellidos: 'Pérez'}) // Error porque falta dni

db.pacientes.insert({nombre: 'Juan', apellidos: 'Pérez', dni: 8648364}) // Error de tipo en dni

db.pacientes.insert({nombre: 'Juan', apellidos: 'Pérez', dni: '864836466A'}) // Ok

Con esta configuración sigue permitiendo propiedades que no existan, para forzar a solo propiedades
conocidas añadir:

A nivel de jsonSchema

additionalProperties: false

A nivel de properties

_id: <tipo-que-vaya-a-tener-id>

Por ejemplo:

db.createCollection("pacientes", {
    validator: {
        $jsonSchema: {
            bsonType: "object",
            required: ["nombre","apellidos","dni"],
            additionalProperties: false,
            properties: {
                _id: {
                    bsonType: "objectId"
                },
                nombre: {
                    bsonType: "string",
                    description: "debe ser strig y es obligatorio"
                },
                apellidos: {
                    bsonType: "string",
                    description: "debe ser strig y es obligatorio"
                },
                dni: {
                    bsonType: "string",
                    description: "debe ser strig y es obligatorio"
                },
                edad: {
                    bsonType: ["number","null"],
                    minimum: 0,
                    maximum: 120,
                    description: "debe ser number entre 0 y 120 ó null" 
                },
                direccion: {
                    bsonType: "object",
                    properties: {
                        calle: {
                            bsonType: "string",
                            description: "debe ser strig"
                        },
                        cp: {
                            bsonType: "string",
                            description: "debe ser strig"
                        },
                        localidad: {
                            bsonType: "string",
                            enum: ["Sevilla","Málaga","Cádiz"],
                            description: "debe ser Sevilla, Málaga o Cádiz"
                        }
                    }
                }
            }
        }
    }
})

db.pacientes.insert({nombre: 'Juan', apellidos: 'Pérez', dni: '864836466A', dolencia: 'Renal'}) // Error validacion

Ver validaciones

db.getCollectionInfos({name:'pacientes'}) // En la shell clásica de mongo


Para colecciones ya existentes en vez de utilizar createCollection se usa runCommand() 

db.runCommand({
    collMod: <nombre-coleccion>,
    validator: <validación>,
    validationLevel: <strict | moderate>
})

// Si le pasamos validationLevel: 'moderate' permite implementar la validación aunque haya registros
que la incumplan y se pondrá en marcha con los nuevos documentos y con las actualizaciones de los
antiguos. Si no se le pasa validationLevel y hay docs que inclumplen la validación no se podrá implementar

