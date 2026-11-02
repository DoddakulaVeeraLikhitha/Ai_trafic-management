
import streamlit as st

# ---------------------------------------
# AI-Based Adaptive Traffic Signal Agent
# ---------------------------------------

st.set_page_config(
    page_title="AI Traffic Management Agent",
    page_icon="🚦",
    layout="wide"
)

st.title("🚦 AI-Based Adaptive Traffic Signal Management")
st.subheader("Using Search and Rule-Based Decision Making")

st.write(
    "This website simulates an intelligent traffic agent "
    "that selects a road based on traffic density and waiting time."
)

st.info(
    "Simulation only: this project does not control real traffic signals."
)

# Road input
st.sidebar.header("Traffic Data Input")

roads = ["North", "South", "East", "West"]
traffic_data = {}

for road in roads:
    st.sidebar.subheader(road + " Road")

    vehicles = st.sidebar.number_input(
        f"Vehicles on {road} Road",
        min_value=0,
        max_value=200,
        value=10,
        step=1,
        key=f"vehicles_{road}"
    )

    waiting_time = st.sidebar.number_input(
        f"Waiting time on {road} (seconds)",
        min_value=0,
        max_value=600,
        value=30,
        step=5,
        key=f"waiting_{road}"
    )

    emergency = st.sidebar.checkbox(
        f"Emergency vehicle on {road}",
        key=f"emergency_{road}"
    )

    traffic_data[road] = {
        "vehicles": vehicles,
        "waiting": waiting_time,
        "emergency": emergency
    }

# Rule-based traffic classification
def classify_traffic(vehicles):
    if vehicles >= 40:
        return "High"
    elif vehicles >= 15:
        return "Medium"
    else:
        return "Low"


# Search-based agent decision
def choose_road(data):
    candidates = []

    for road, info in data.items():
        # Rule: an emergency vehicle receives highest priority
        if info["emergency"]:
            priority = 10000 + info["waiting"]
        else:
            # Search score based on congestion and waiting time
            priority = (
                info["vehicles"] * 2
                + info["waiting"] * 0.5
            )

        candidates.append({
            "road": road,
            "vehicles": info["vehicles"],
            "waiting": info["waiting"],
            "emergency": info["emergency"],
            "priority": priority
        })

    # Search all candidate roads and select the highest score.
    # If scores tie, the first road in the displayed order wins.
    candidates.sort(
        key=lambda item: item["priority"],
        reverse=True
    )

    return candidates[0], candidates


# Adaptive duration for simulation
def calculate_green_time(vehicles):
    duration = 15 + vehicles
    return min(max(duration, 15), 60)


# Run the agent
if st.button("▶ Analyze Traffic and Select Signal", type="primary"):

    selected, ranked_roads = choose_road(traffic_data)

    selected_road = selected["road"]
    vehicle_count = selected["vehicles"]
    waiting = selected["waiting"]

    green_time = calculate_green_time(vehicle_count)
    traffic_level = classify_traffic(vehicle_count)

    st.success("Traffic analysis completed!")

    # Main result
    st.header("🤖 AI Agent Decision")

    col1, col2, col3 = st.columns(3)

    col1.metric("Selected Road", selected_road)
    col2.metric("Suggested Green Time", f"{green_time} seconds")
    col3.metric("Traffic Level", traffic_level)

    st.markdown("---")
    st.subheader("🚦 Simulated Signal Status")

    for road in roads:
        if road == selected_road:
            st.markdown(f"🟢 **{road} Road: GREEN**")
        else:
            st.markdown(f"🔴 **{road} Road: RED**")

    st.markdown("---")
    st.subheader("🔍 Search Results and Traffic Analysis")

    # Display the ranking of all roads
    for rank, item in enumerate(ranked_roads, start=1):
        level = classify_traffic(item["vehicles"])

        st.write(
            f"**{rank}. {item['road']} Road** — "
            f"{item['vehicles']} vehicles, "
            f"{item['waiting']} seconds waiting, "
            f"{level} traffic"
        )

    st.markdown("---")
    st.subheader("🧠 Explanation of the Decision")

    if selected["emergency"]:
        st.write(
            f"The agent selected {selected_road} Road because "
            "an emergency vehicle was reported there. "
            "Emergency priority is a rule in this simulation."
        )
    else:
        st.write(
            f"The agent selected {selected_road} Road because "
            "it received the highest score from the traffic search."
        )

    st.write(
        "The score considers the number of waiting vehicles "
        "and their waiting time. The green duration is then "
        "calculated using the vehicle count."
    )

    st.caption(
        "This is an educational model, not a validated traffic "
        "control algorithm. Real traffic systems require "
        "safety constraints, sensor validation, and expert testing."
    )

else:
    st.warning(
        "Enter traffic values in the left sidebar, then click "
        "'Analyze Traffic and Select Signal'."
    )

st.markdown("---")
st.caption(
    "Student AI Capstone Project | Search + Rule-Based Agent"
)
