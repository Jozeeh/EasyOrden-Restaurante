<template>
    <ion-page>
        <ion-header>
            <ion-toolbar class="bgcolor-header">
                <ion-buttons slot="end">
                    <ion-menu-button color="light"></ion-menu-button>
                </ion-buttons>
                <ion-title>Pedidos a preparar</ion-title>
            </ion-toolbar>
        </ion-header>

        <ion-content class="fondo">
            <ion-grid>
                <ion-row>
                    <ion-col class="ion-text-center">
                        <ion-card style="--background: white;">
                            <img src="/productos-preparar.webp">
                            <ion-card-header>
                                <ion-card-title style="color: black;">Pedidos a preparar</ion-card-title>
                            </ion-card-header>
                        </ion-card>
                    </ion-col>
                </ion-row>

                <ion-row v-if="cargandoProductos == true">
                    <ion-col>
                        <ion-card style="background-color: white;">
                            <ion-card-header>
                                <ion-card-title style="color: black;" class="ion-text-center">Cargando productos...</ion-card-title>
                            </ion-card-header>

                            <div
                                style="height: 100%; display: flex; justify-content: center;align-items: center; padding-bottom: 15px;">
                                <ion-spinner name="circular" style="--color: black;"></ion-spinner>
                            </div>
                        </ion-card>
                    </ion-col>
                </ion-row>

                <ion-row v-else>
                    <ion-col>
                        <!-- ION ACORDEON -->
                        <ion-accordion-group v-for="(pedidoClientes, i) in pedidos2" :key="i">
                            <ion-accordion value="first" :toggle-icon="caretDownCircle" toggle-icon-slot="end"
                                class="accordion-icon item-description">
                                <ion-item slot="header" class="item-producto">
                                    <ion-label>{{ pedidoClientes.name }}</ion-label>
                                </ion-item>
                                <div class="ion-padding accordionExpand" slot="content">
                                    <!-- PRODUCTOS DE CADA CLIENTE -->
                                    <ion-card v-for="(pedidos, i) in pedidoClientes.pedidos" style="background-color: white; color: black;">
                                        <ion-card-header>
                                            <div><b>Cliente:</b> {{ pedidos.name }}</div>
                                            <div><b>El pedido fue hecho:</b> {{ pedidos.created_at }}</div>
                                            <ion-item style="--background: white; color: black;">
                                                <ion-thumbnail slot="end"><img alt="pedido-imagen"
                                                        :src="pedidos.imagen" /></ion-thumbnail>
                                                <ion-label> {{ pedidos.nombrePlato }}</ion-label>
                                            </ion-item>
                                            <ion-card-title style="color: black;">{{ pedidos.numMesa }}</ion-card-title>
                                        </ion-card-header>

                                        <ion-card-content>
                                            
                                            <b>El producto está en:</b> {{ pedidos.estadoPedido }}
                                            <br>

                                            <!-- Si el pedido es recien creado -->
                                            <ion-button v-if="pedidos.estadoPedido == 'Ordenado'" color="warning" @click="actualizarEstadoPedido(pedidos.idPedido, 'En preparación')">Marcar en
                                                preparación</ion-button>

                                            <ion-button v-if="pedidos.estadoPedido == 'En preparación'"
                                                color="primary" @click="actualizarEstadoPedido(pedidos.idPedido, 'Listo')">Marcar listo</ion-button>

                                            <ion-button v-if="pedidos.estadoPedido == 'Entregado'" color="success"
                                                disabled>ENTREGADO!</ion-button>
                                        </ion-card-content>
                                    </ion-card>
                                </div>
                            </ion-accordion>
                        </ion-accordion-group>
                    </ion-col>
                </ion-row>
            </ion-grid>
        </ion-content>
    </ion-page>
</template>


<script>
import { IonButtons, IonContent, IonHeader, IonMenuButton, IonPage, IonTitle, IonToolbar, IonGrid, IonRow, IonCol, IonCard, IonCardHeader, IonCardTitle, IonCardSubtitle, IonCardContent, IonIcon, IonButton, IonSelect, IonSelectOption, IonFooter, IonItem, IonAccordionGroup, IonAccordion, IonLabel, IonThumbnail, IonSpinner } from '@ionic/vue';

import axios from 'axios';

export default {
    name: 'InicioPage',
    components: {
        IonButtons, IonContent, IonHeader, IonMenuButton, IonPage, IonTitle, IonToolbar, IonGrid, IonRow, IonCol, IonCard, IonCardHeader, IonCardTitle, IonCardSubtitle, IonCardContent, IonIcon, IonButton, IonSelect, IonSelectOption, IonFooter, IonItem, IonAccordionGroup, IonAccordion, IonLabel, IonThumbnail, IonSpinner
    },
    data() {
        return {
            ipLocal: this.$store.state.ipLocal,
            categoriaEstadoPedido: 'Ordenado',
            pedidos: {},
            pedidos2: {},
            cargandoProductos: false,
            config: {}
        }
    },
    methods: {
        obtenerPedidos() {
            this.cargandoProductos = true
            axios.get(`${this.ipLocal}/pedidos/select`, this.config)
                .then(response => {
                    this.cargandoProductos = false
                    this.pedidos2 = response.data.data
                    console.log(response.data.data);
                })
                //Si ocurre un error se imprimirá en consola
                .catch(error => {
                    console.error('OCURRIO UN ERROR: ', error);
                })
        },
        // Obtenemos los pedidos que se han hecho con los detalles del plato
        obtenerPedidosCategoria() {
            axios.get(`${this.ipLocal}/pedidos/select/${this.categoriaEstadoPedido}`, this.config)
                .then(response => {
                    this.pedidos = response.data.data
                    console.log(response.data.data);
                })
                //Si ocurre un error se imprimirá en consola
                .catch(error => {
                    console.error('OCURRIO UN ERROR: ', error);
                })
        },
        actualizarEstadoPedido(idPedido, nuevoEstado) {

            if(nuevoEstado == "En preparación"){
                nuevoEstado = "En preparación"

            } else if(nuevoEstado == "Listo"){
                nuevoEstado = "Entregado"
            }

            const data = {
                estadoPedido: nuevoEstado
            };

            axios.put(`${this.ipLocal}/pedidos/update/${idPedido}`, data, this.config)
                .then(response => {
                    console.log(response.data.data);
                    this.obtenerPedidos()
                })
                //Si ocurre un error se imprimirá en consola
                .catch(error => {
                    console.error('OCURRIO UN ERROR: ', error);
                })
        },
        // Obtenemos datos del usuario con Ionic/Storage
        obtenerDatosUsuario() {
            try {
                // Usa la función get para recuperar los datos del usuario por su clave
                this.$storage.get('tokenInicioSesion')
                .then(userData => {
                    if (userData) {
                        // userData contiene los datos del usuario
                        console.log('Datos del usuario:', userData);
                        this.$store.state.datosUsuario = userData;
                    } else {
                        console.log('No se encontraron datos de usuario.');
                    }
                })
                .catch(error => {
                    console.error('Error al recuperar datos del usuario:', error);
                });
            } catch (error) {
                console.error('Error al recuperar datos del usuario:', error);
            }
        },
        // Obtenemos token
        async getToken() {
            let token = await this.$storage.get('easyToken')
            this.config = {
                headers: {
                    'Authorization': 'Bearer ' + token
                }
            }

            this.obtenerPedidos()
        }
    },
    beforeCreate(){
        //Verificar si ya tenemos una sesión iniciada
        this.$storage.get('tokenInicioSesion')
            .then(token => {
                if (!token) {
                    //Si no tenemos sesión iniciada
                    console.log('Inicia sesión o registrate!')
                } else {
                    // Si se encuentra un token, obtiene los datos del usuario
                    this.obtenerDatosUsuario();
                }
            })
            .catch(error => {
                console.error('Error al verificar la sesión:', error);
            });
    },
    ionViewWillEnter() {
        this.getToken()
    }
}
</script>


<style scoped>
/* ACCORDION STYLE */
ion-accordion-group {
    margin: 0 auto;
    width: 94%;
}
.item-producto {
    --background: rgb(227, 227, 227);
    --color: black;
    color: black;
}
.item-description {
    background: #fbfcff;
    color: black;
}

.acordionExpand {
    /* border-bottom-left-radius: 10px;
    border-bottom-right-radius: 10px; */
    background: #fbfcff;
    --color: black;
}

/* FONDO ESTILO */
.fondo {
    --background: none;
    background-image: url('/FondoSesion.webp');

    background-position: center center;
    background-repeat: no-repeat;
    background-size: cover;
}

.fondo::before {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(14, 14, 14, 0.5);
}

/* ESTILOS HEADER */
.bgcolor-header {
    --background: #c93e4f;
    --color: white;
}

.contenedor {
    /* border: 1px solid black; */
    width: 90%;
    margin: 0 auto;
}
</style>