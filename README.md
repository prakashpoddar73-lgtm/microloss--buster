import datetime
import json
st.set_page_config(page_title="Microloss Buster Dashboard", page_icon=":bar_chart:", layout="wide")
import smtplib
from email.mime.text import MIMEText
import uuid
import streamlit as st

# Configure secure file logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s [%(levelname)s] %(message)s',
    handlers=[
        logging.FileHandler("security_audit.log"),
        logging.StreamHandler()
    ]
)
# =====================================================================# AUTOMATED EMAIL ALERT LAYER# =====================================================================def send_instant_email_alert(alert_type: str, alert_message: str):"""""
def send_instant_email_alert(alert_type: str, alert_message: str):
    logging.info(f"📧 [EMAIL MOCK ALERT]: Type: {alert_type} | Details: {alert_message}")
    return

    sender_email = "your_security_alerts@gmail.com"
    sender_password = "your_app_password"          
    owner_email = "owner_personal_email@gmail.com"
    
    if sender_email == "your_security_alerts@gmail.com":
        logging.info(f"📧 [EMAIL MOCK]: Alert triggered but credentials not set. Message: {alert_message}")
        pass
        exit()

    subject = f"🚨 [SECURITY BREACH]: {alert_type} Detected at MicroLoss Buster"
    body = f"""
    Attention Store Owner,

    MicroLoss Buster's AI Anomaly Engine has detected an active financial leak or theft pattern.

    [ALERT PROFILE]: {alert_type}
    [TIMESTAMP]: {datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')}
    [DETAILS]: {alert_message}

    Please review your Admin Dashboard immediately.

    -- Securely sent by MicroLoss Buster AI Engine
    """
    
    msg = MIMEText(body)
    msg['Subject'] = subject
    msg['From'] = sender_email
    msg['To'] = owner_email

    try:
        with smtplib.SMTP_SSL("://gmail.com", 465) as server:
            server.login(sender_email, sender_password)
            server.sendmail(sender_email, owner_email, msg.as_string())
        logging.info(f"🚀 [EMAIL SUCCESS]: Instant alert dispatched to {owner_email}")
    except Exception as e:
        logging.error(f"❌ [EMAIL FAILURE]: Critical transmission error. Details: {str(e)}")

# =====================================================================# CORE ENGINE STATE CLASS# =====================================================================class AIAntiFraudInventorySystem:
    def __init__(self):
        # Initialize Persistent Application States using Streamlit Session State
        if "inventory" not in st.session_state:
            st.session_state.inventory = {"Premium Coffee": 100, "Artisan Chocolate": 150}
        if "prices" not in st.session_state:
            st.session_state.prices = {"Premium Coffee": 5.00, "Artisan Chocolate": 3.50}
        if "employee_registry" not in st.session_state:
            st.session_state.employee_registry = {
                "EMP_01": {"name": "Alice Smith", "role": "Cashier", "is_flagged": False},
                "EMP_02": {"name": "Bob Jones", "role": "Cashier", "is_flagged": False},
                "EMP_03": {"name": "Charlie Rogue", "role": "Seasonal Staff", "is_flagged": False}
            }
        if "is_premium_tier" not in st.session_state:
            st.session_state.is_premium_tier = False
        if "premium_expiry" not in st.session_state:
            st.session_state.premium_expiry = None
        if "transaction_logs" not in st.session_state:
            st.session_state.transaction_logs = []
        if "security_alerts" not in st.session_state:
            st.session_state.security_alerts = []

    def activate_premium_ai(self):
        st.session_state.is_premium_tier = True
        st.session_state.premium_expiry = (datetime.datetime.now() + datetime.timedelta(days=365)).strftime("%Y-%m-%d")
        logging.info("🔓 [SYSTEM]: AI Engine upgraded securely via external platform hook.")

    def process_transaction(self, employee_id: str, action_type: str, item: str, quantity: int, cash_registered: float = 0.0) -> bool:
        if employee_id not in st.session_state.employee_registry:
            st.error(f"Transaction Denied: Employee ID '{employee_id}' is not registered.")
            return False
        if item not in st.session_state.inventory:
            st.error(f"Transaction Denied: Item '{item}' does not exist in catalog.")
            return False
        if quantity <= 0:
            st.error("Transaction Denied: Invalid quantity scale.")
            return False

        tx_id = str(uuid.uuid4())[:8]
        timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        original_stock = st.session_state.inventory[item]

        try:
            if action_type in ["sale", "micro_loss", "free_sample"]:
                if original_stock < quantity:
                    st.warning(f"⚠️ [STOCK LEAK BLOCKED]: Insufficient stock for {item}.")
                    return False
                st.session_state.inventory[item] -= quantity

            log_entry = {
                "tx_id": tx_id, "time": timestamp, "emp": employee_id, 
                "type": action_type, "item": item, "qty": quantity, "cash": float(cash_registered)
            }
            st.session_state.transaction_logs.append(log_entry)

            if st.session_state.is_premium_tier:
                self._run_anomaly_detection_engine(log_entry)
            return True
        except Exception as e:
            st.session_state.inventory[item] = original_stock
            logging.critical(f"🚨 [ROLLBACK]: State corruption error: {str(e)}")
            return False

    def _run_anomaly_detection_engine(self, current_log: dict):
        emp_id = current_log["emp"]
        action = current_log["type"]
        item = current_log["item"]
        qty = current_log["qty"]
        cash = current_log["cash"]
        emp_name = st.session_state.employee_registry[emp_id]["name"]
        
        # Rule 1: Cash Intake Mismatch Detection
        if action == "sale":
            expected_cash = st.session_state.prices[item] * qty
            if cash < expected_cash:
                variance = expected_cash - cash
                alert_msg = f"{emp_name} ({emp_id}) entered ${cash:.2f} for {qty}x {item}. Expected: ${expected_cash:.2f}. Deficit: -${variance:.2f}."
                st.session_state.security_alerts.append({"time": current_log["time"], "type": "CASH_MISMATCH", "msg": alert_msg})
                st.session_state.employee_registry[emp_id]["is_flagged"] = True
                send_instant_email_alert("CASH_MISMATCH", alert_msg)

        # Rule 2: Micro-Loss Fraud Spike Tracking
        if action == "micro_loss":
            recent_losses = [
                log for log in st.session_state.transaction_logs[-10:]
                if log["emp"] == emp_id and log["type"] == "micro_loss"
            ]
            if len(recent_losses) >= 3:
                alert_msg = f"{emp_name} ({emp_id}) logged {len(recent_losses)} continuous micro-losses in current shift."
                st.session_state.security_alerts.append({"time": current_log["time"], "type": "MICRO_LOSS_SPIKE", "msg": alert_msg})
                st.session_state.employee_registry[emp_id]["is_flagged"] = True
                send_instant_email_alert("MICRO_LOSS_SPIKE", alert_msg)

# =====================================================================# THE VISUAL USER INTERFACE (STREAMLIT FRAMEWORK)# =====================================================================
st.set_page_config(page_title="Microloss Buster Dashboard", page_icon="⚙️"engine = AIAntiFraudInventorySystem()

st.title("🛡️ MicroLoss Buster: Enterprise Owner Interface")
st.markdown("---")
col_left, col_right = st.columns()
with col_left:
    st.header("⚙️ Settings & Configuration")
    
    with st.expander("💳 Software License Access"):
        if st.session_state.is_premium_tier:
            st.success(f"🟢 Premium AI Active until {st.session_state.premium_expiry}")
        else:
            st.info("🔴 Currently running Free Tier (AI deactivated)")
            if st.button("Unlock Premium AI Engine ($2/yr)"):
                engine.activate_premium_ai()
                st.rerun()

    with st.expander("📦 Add New Catalog Products"):
        new_item = st.text_input("Product Name", placeholder="e.g., Cold Brew Coffee")
        new_stock = st.number_input("Starting Warehouse Stock", min_value=1, value=100)
        new_price = st.number_input("Target Unit Price ($)", min_value=0.1, value=4.50)
        if st.button("Commit Product to System Matrix"):
            if new_item:
                st.session_state.inventory[new_item] = new_stock
                st.session_state.prices[new_item] = new_price
                st.success(f"Successfully added '{new_item}' to system!")
                st.rerun()

    with st.expander("👥 Register Store Personnel"):
        new_emp_id = st.text_input("New Employee Access ID Token", placeholder="e.g., EMP_05")
        new_emp_name = st.text_input("Full Legal Name")
        new_emp_role = st.selectbox("Company Designation Role", ["Cashier", "Store Manager", "Seasonal Staff"])
        if st.button("Assign System Access Credentials"):
            if new_emp_id and new_emp_name:
                st.session_state.employee_registry[new_emp_id] = {"name": new_emp_name, "role": new_emp_role, "is_flagged": False}
                st.success(f"Access granted for {new_emp_name}!")
                st.rerun()

    with st.expander("🛒 Simulate Live Shop Traffic"):
        sim_emp = st.selectbox("Select Active Staff Member", list(st.session_state.employee_registry.keys()))
        sim_action = st.selectbox("Action Execution Type", ["sale", "micro_loss", "cart_deletion"])
        sim_item = st.selectbox("Target Product", list(st.session_state.inventory.keys()))
        sim_qty = st.number_input("Quantity Traded", min_value=1, value=1)
        sim_cash = st.number_input("Actual Cash Taken ($)", min_value=0.0, value=0.0)
        
if st.button("Execute Counter Action"):
    success = engine.process_transaction(sim_emp, sim_action, sim_item, sim_qty, sim_cash)
    if success:
        st.success("Action evaluated and logged.")
        st.rerun()
with col_right:
	st.header("📊 Real-Time Operations Telemetry")
	revenue_calc = sum([log["cash"] for log in st.session_state.transaction_logs if log["type"] == "sale"])
	metric_col1, metric_col2, metric_col3 = st.columns(3)
	metric_col1.metric("💵 Valid Cash Revenue Verified", f"${revenue_calc:.2f}")
	metric_col2.metric("🚨 Active AI System Flags", len(st.session_state.security_alerts))
	metric_col3.metric("📦 Active Catalog Types", len(st.session_state.inventory))
	st.subheader("🚨 Live Anti-Theft Incident Log")
    if not st.session_state.security_alerts:
            	st.write("✅ System status secure. No behavioral anomalies recorded.")
    else:
	    
        	 for alert in reversed(st.session_state.security_alerts):
	     		st.error(f"[{alert['time']}] [{alert['type']}] -> {alert['msg']}")
		
     st.subheader("📦 Live Warehouse Stock Balances")
     for prod, qty in st.session_state.inventory.items():
         st.progress(min(max(qty / 200.0, 0.0), 1.0), text=f"{prod}: {qty} units remaining (${st.session_state.prices[prod]:.2f}/ea)")
		
      st.subheader("👥 Active Staff Access Profiles")
      for idx, (eid, data) in enumerate(st.session_state.employee_registry.items()):
	  status_txt = "⚠️ HIGH RISK ASSIGNED" if data["is_flagged"] else "✅ Nominal / Clear"
	  st.text(f"[{eid}] Name: {data['name'].ljust(15)} | Role: {data['role'].ljust(12)} | Profile: {status_txt}")


