# Prototipo
Gordazzo
import streamlit as st
import pandas as pd
import plotly.express as px

# 1. Configurazione Pagina
st.set_page_config(page_title="Monitoraggio Cetacei", layout="wide")

# 2. Controllo Accesso (Password)
st.sidebar.title("Area Riservata")
password = st.sidebar.text_input("Inserisci Password", type="password")

# Sostituisci 'balena123' con la tua password preferita
if password == "balena123": 
    st.title("🐋 Dashboard Avvistamenti Cetacei")
    
    # 3. Caricamento del TUO file Excel
    # Assicurati che il file si chiami 'dati.xlsx' su GitHub
    try:
        df = pd.read_excel("dati.xlsx")
        
        # 4. Filtri nella barra laterale
        st.sidebar.header("Filtri")
        if 'Specie' in df.columns:
            lista_specie = df['Specie'].unique()
            scelta = st.sidebar.multiselect("Seleziona Specie", options=lista_specie, default=lista_specie)
            df_filtrato = df[df['Specie'].isin(scelta)]
        else:
            df_filtrato = df
            st.error("Errore: Non trovo la colonna 'Specie' nel tuo Excel.")

        # 5. Indicatori veloci
        c1, c2 = st.columns(2)
        c1.metric("Numero Avvistamenti", len(df_filtrato))
        if 'Esemplari' in df.columns:
            c2.metric("Totale Individui", df_filtrato['Esemplari'].sum())

        # 6. LA MAPPA
        st.subheader("Mappa Geografica")
        # Cambia 'Latitudine' e 'Longitudine' se i nomi nel tuo Excel sono diversi
        if 'Latitudine' in df.columns and 'Longitudine' in df.columns:
            fig = px.scatter_mapbox(df_filtrato, lat="Latitudine", lon="Longitudine", 
                                    color="Specie" if 'Specie' in df.columns else None,
                                    hover_name="Specie" if 'Specie' in df.columns else None,
                                    zoom=5, height=600)
            fig.update_layout(mapbox_style="open-street-map", margin={"r":0,"t":0,"l":0,"b":0})
            st.plotly_chart(fig, use_container_width=True)
        else:
            st.warning("Per vedere la mappa, il tuo Excel deve avere le colonne 'Latitudine' e 'Longitudine'.")

        # 7. Tabella Dati
        with st.expander("Vedi i dati grezzi"):
            st.write(df_filtrato)

    except Exception as e:
        st.error(f"Non riesco a leggere il file Excel. Assicurati di aver caricato 'dati.xlsx' su GitHub. Errore: {e}")

else:
    st.info("Inviaci la password corretta per visualizzare la mappa e i dati dell'associazione.")
