# Prototipo
Gordazzo
import streamlit as st
import pandas as pd
import numpy as np
import plotly.express as px
import datetime

# ==========================================
# 1. CONFIGURAZIONE PAGINA (Sito Web)
# ==========================================
st.set_page_config(
    page_title="Area Riservata | Monitoraggio Cetacei Mediterraneo",
    page_icon="🐋",
    layout="wide", # Usa tutto lo schermo
    initial_sidebar_state="expanded"
)

# Messaggio di benvenuto pubblico
st.title("🐋 Portale Avvistamenti Cetacei - Associazione Mare Nostrum")
st.markdown("---")

# ==========================================
# 2. SISTEMA DI SICUREZZA (Password)
# ==========================================
st.sidebar.image("https://upload.wikimedia.org/wikipedia/commons/7/7b/Logo_WWF_Italia.png", width=100) # Logo esempio
st.sidebar.title("Accesso Membri")
password = st.sidebar.text_input("Inserisci la Password dell'Associazione", type="password")

# --- LA PASSWORD È: cetacei2024 ---
if password == "cetacei2024":
    st.sidebar.success("Accesso effettuato!")
    
    # ==========================================
    # 3. GENERAZIONE DATI CASUALI (Fake Data)
    # ==========================================
    @st.cache_data # Mantiene i dati in memoria per velocità
    def genera_dati_fake():
        # Creiamo 150 avvistamenti finti
        num_avvistamenti = 150
        
        species_list = ['Stenella striata', 'Tursiope', 'Balenottera comune', 'Capodoglio', 'Grampo', 'Globicefalo']
        associa_list = ['MArHE Center', 'WWF Italia', 'Tethys Research Institute', 'Mariasole']

        # Generiamo coordinate casuali nel Mediterraneo centrale (tra Italia e Grecia)
        lats = np.random.uniform(36.0, 43.5, num_avvistamenti)
        lons = np.random.uniform(7.5, 20.0, num_avvistamenti)
        
        # Generiamo date casuali nell'ultimo anno
        start_date = datetime.date(2023, 1, 1)
        end_date = datetime.date(2023, 12, 31)
        time_between_dates = end_date - start_date
        days_between_dates = time_between_dates.days
        random_dates = [start_date + datetime.timedelta(days=np.random.randint(days_between_dates)) for _ in range(num_avvistamenti)]

        # Creiamo il DataFrame (come se fosse il tuo Excel)
        data = pd.DataFrame({
            'Data': random_dates,
            'Specie': np.random.choice(species_list, num_avvistamenti),
            'Latitudine': lats,
            'Longitudine': lons,
            'Numero_Individui': np.random.randint(1, 25, num_avvistamenti),
            'Associazione_Rilevante': np.random.choice(associa_list, num_avvistamenti),
            'Profondita_Fondale_m': np.random.randint(50, 2500, num_avvistamenti)
        })
        # Ordiniamo per data
        data = data.sort_values(by='Data')
        return data

    # Carichiamo i dati generati
    df = genera_dati_fake()

    # ==========================================
    # 4. FILTRI INTERATTIVI (Sidebar)
    # ==========================================
    st.sidebar.header("Filtra Mappa e Statistiche")
    
    # Filtro Specie (Multiselect)
    all_species = sorted(df['Specie'].unique())
    selected_species = st.sidebar.multiselect(
        "Seleziona Specie",
        options=all_species,
        default=all_species[:3] # Preselezioniamo le prime 3 per far scena
    )

    # Filtro Numero Individui (Slider)
    max_individui = int(df['Numero_Individui'].max())
    min_individui, max_individui_sel = st.sidebar.slider(
        "Numero Individui nel gruppo",
        0, max_individui, (0, max_individui)
    )

    # Filtro Data (Date Input)
    min_date = df['Data'].min()
    max_date = df['Data'].max()
    date_range = st.sidebar.date_input(
        "Periodo di osservazione",
        value=(min_date, max_date),
        min_value=min_date,
        max_value=max_date
    )

    # --- APPLICAZIONE FILTRI ---
    # Gestione filtro data (verifica che siano selezionate entrambe le date)
    if len(date_range) == 2:
        start_filter, end_filter = date_range
    else:
        start_filter, end_filter = min_date, max_date

    mask = (
        (df['Specie'].isin(selected_species)) &
        (df['Numero_Individui'] >= min_individui) &
        (df['Numero_Individui'] <= max_individui_sel) &
        (df['Data'] >= start_filter) &
        (df['Data'] <= end_filter)
    )
    df_filtered = df[mask]

    # ==========================================
    # 5. DASHBOARD - KPI (Indicatori Chiave)
    # ==========================================
    st.header("Statistiche Rapide (Dati Filtrati)")
    kpi1, kpi2, kpi3, kpi4 = st.columns(4)
    
    with kpi1:
        st.metric(label="Totale Avvistamenti", value=len(df_filtered))
    with kpi2:
        st.metric(label="Esemplari Totali", value=df_filtered['Numero_Individui'].sum())
    with kpi3:
        st.metric(label="Profondità Media (m)", value=int(df_filtered['Profondita_Fondale_m'].mean()))
    with kpi4:
        st.metric(label="Associazione più attiva", value=df_filtered['Associazione_Rilevante'].mode()[0] if not df_filtered.empty else "N/A")

    st.markdown("---")

    # ==========================================
    # 6. DASHBOARD - MAPPA INTERATTIVA
    # ==========================================
    st.subheader("Mappa Geografica degli Avvistamenti")
    
    if not df_filtered.empty:
        # Creiamo una mappa professionale con Plotly
        fig_map = px.scatter_mapbox(
            df_filtered,
            lat="Latitudine",
            lon="Longitudine",
            color="Specie", # Colore punto base alla specie
            size="Numero_Individui", # Dimensione punto base al numero
            size_max=15,
            hover_name="Specie", # Titolo popup
            hover_data=["Data", "Numero_Individui", "Associazione_Rilevante", "Profondita_Fondale_m"], # Dettagli popup
            zoom=5,
            height=600,
            color_discrete_sequence=px.colors.qualitative.Bold # Palette colori visibile
        )
        
        # Impostiamo lo stile della mappa (OpenStreetMap è gratis)
        fig_map.update_layout(
            mapbox_style="open-street-map",
            margin={"r":0,"t":0,"l":0,"b":0}, # Rimuove i margini bianchi
            legend=dict(yanchor="top", y=0.99, xanchor="left", x=0.01) # Posizione legenda
        )
        
        # Mostriamo la mappa
        st.plotly_chart(fig_map, use_container_width=True)
    else:
        st.warning("Nessun dato corrisponde ai filtri selezionati. Prova ad allargare la selezione.")

    st.markdown("---")

    # ==========================================
    # 7. DASHBOARD - GRAFICI STATISTICI
    # ==========================================
    st.subheader("Analisi Statistica")
    col_chart1, col_chart2 = st.columns(2)

    with col_chart1:
        st.markdown("**Distribuzione Avvistamenti per Specie**")
        if not df_filtered.empty:
            # Grafico a barre
            fig_bar = px.bar(
                df_filtered.groupby('Specie').size().reset_index(name='Conteggio'),
                x='Specie',
                y='Conteggio',
                color='Specie',
                color_discrete_sequence=px.colors.qualitative.Bold
            )
            fig_bar.update_layout(showlegend=False)
            st.plotly_chart(fig_bar, use_container_width=True)

    with col_chart2:
        st.markdown("**Avvistamenti nel Tempo (Andamento Mensile)**")
        if not df_filtered.empty:
            # Grafico temporale
            df_filtered['Mese'] = df_filtered['Data'].dt.to_period('M').astype(str)
            df_timeline = df_filtered.groupby('Mese').size().reset_index(name='Conteggio')
            fig_time = px.line(
                df_timeline,
                x='Mese',
                y='Conteggio',
                markers=True
            )
            st.plotly_


        
