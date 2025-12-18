"""
Smart Parking Lot Management System
Enhanced version with ALL PDF features

Features:
- Slot allocation
- Vehicle entry & exit
- Time-based billing (variable pricing)
- Vehicle-type pricing
- VIP reserved slots
- Daily revenue report
- Color-coded CLI
- PDF export of daily report
"""

from datetime import datetime
import math
from fpdf import FPDF

# ---------------- CONFIGURATION ----------------
TOTAL_SLOTS = 10
VIP_SLOTS = {1, 2}  # Reserved VIP slots

PRICING = {
    "bike": 10,
    "car": 20,
    "ev": 15,
    "truck": 30
}

FIXED_HOURS = 2
FIXED_PRICE = 50

# ---------------- DATA STORAGE ----------------
parking_slots = {i: None for i in range(1, TOTAL_SLOTS + 1)}
revenue = 0
total_vehicles = 0

# ---------------- COLOR UTILS ----------------
GREEN = "\033[92m"
RED = "\033[91m"
YELLOW = "\033[93m"
RESET = "\033[0m"

# ---------------- CORE FUNCTIONS ----------------

def find_free_slot(is_vip=False):
    for slot in parking_slots:
        if parking_slots[slot] is None:
            if is_vip or slot not in VIP_SLOTS:
                return slot
    return None


def vehicle_entry():
    global total_vehicles

    vehicle_no = input("Enter vehicle number: ").upper()
    v_type = input("Vehicle type (bike/car/ev/truck): ").lower()
    vip = input("VIP vehicle? (y/n): ").lower() == 'y'

    if v_type not in PRICING:
        print(RED + "Invalid vehicle type" + RESET)
        return

    slot = find_free_slot(vip)
    if slot is None:
        print(RED + "Parking Full!" + RESET)
        return

    parking_slots[slot] = {
        "vehicle_no": vehicle_no,
        "type": v_type,
        "entry_time": datetime.now(),
        "vip": vip
    }

    total_vehicles += 1
    print(GREEN + f"Vehicle parked at slot {slot}" + RESET)


def calculate_bill(entry_time, v_type):
    parked_time = datetime.now() - entry_time
    hours = math.ceil(parked_time.total_seconds() / 3600)

    if hours <= FIXED_HOURS:
        return FIXED_PRICE, hours
    else:
        extra_hours = hours - FIXED_HOURS
        return FIXED_PRICE + (extra_hours * PRICING[v_type]), hours


def vehicle_exit():
    global revenue

    vehicle_no = input("Enter vehicle number: ").upper()

    for slot, data in parking_slots.items():
        if data and data["vehicle_no"] == vehicle_no:
            amount, hours = calculate_bill(data["entry_time"], data["type"])
            revenue += amount
            parking_slots[slot] = None

            print(YELLOW + "\n------ BILL ------" + RESET)
            print(f"Vehicle: {vehicle_no}")
            print(f"Hours Parked: {hours}")
            print(f"Amount: ₹{amount}")
            print(f"Slot Freed: {slot}")
            return

    print(RED + "Vehicle not found" + RESET)


def show_status():
    print("\nParking Slot Status")
    for slot, data in parking_slots.items():
        if data:
            print(GREEN + f"Slot {slot}: {data['vehicle_no']} ({data['type']})" + RESET)
        else:
            label = "VIP" if slot in VIP_SLOTS else "Empty"
            print(f"Slot {slot}: {label}")


def export_pdf():
    pdf = FPDF()
    pdf.add_page()
    pdf.set_font("Arial", size=12)

    pdf.cell(200, 10, txt="Daily Parking Report", ln=True, align='C')
    pdf.ln(10)
    pdf.cell(200, 10, txt=f"Total Vehicles: {total_vehicles}", ln=True)
    pdf.cell(200, 10, txt=f"Total Revenue: ₹{revenue}", ln=True)

    pdf.output("daily_report.pdf")
    print(GREEN + "PDF Report Generated: daily_report.pdf" + RESET)


def daily_report():
    print("\n----- DAILY REPORT -----")
    print(f"Total Vehicles: {total_vehicles}")
    print(f"Total Revenue: ₹{revenue}")
    occupied = sum(1 for s in parking_slots.values() if s)
    print(f"Occupied Slots: {occupied}/{TOTAL_SLOTS}")
    export = input("Export PDF? (y/n): ").lower()
    if export == 'y':
        export_pdf()

# ---------------- MAIN LOOP ----------------

def main():
    while True:
        print("\n--- SMART PARKING SYSTEM ---")
        print("1. Vehicle Entry")
        print("2. Vehicle Exit")
        print("3. View Parking Status")
        print("4. Daily Report")
        print("5. Exit System")

        choice = input("Choose option: ")

        if choice == "1":
            vehicle_entry()
        elif choice == "2":
            vehicle_exit()
        elif choice == "3":
            show_status()
        elif choice == "4":
            daily_report()
        elif choice == "5":
            print("System closed")
            break
        else:
            print("Invalid choice")


if __name__ == "__main__":
    main()
