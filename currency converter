"""
Currency Converter - BCA Mini Project
A simple Tkinter desktop application using live exchange rates from open.er-api.com.
Internet is required for refreshing rates. A cached rate is not used, so the app
shows an error rather than silently presenting old rates as current.
"""

import json
import threading
import urllib.request
import urllib.error
from datetime import datetime
import tkinter as tk
from tkinter import ttk, messagebox

API_URL = "https://open.er-api.com/v6/latest/{}"
TIMEOUT_SECONDS = 12

CURRENCIES = {
    "INR": "Indian Rupee (INR)",
    "USD": "US Dollar (USD)",
    "EUR": "Euro (EUR)",
    "GBP": "British Pound (GBP)",
    "JPY": "Japanese Yen (JPY)",
    "AUD": "Australian Dollar (AUD)",
    "CAD": "Canadian Dollar (CAD)",
    "CHF": "Swiss Franc (CHF)",
    "CNY": "Chinese Yuan (CNY)",
    "SGD": "Singapore Dollar (SGD)",
    "AED": "UAE Dirham (AED)",
    "NZD": "New Zealand Dollar (NZD)",
    "BDT": "Bangladeshi Taka (BDT)",
    "NPR": "Nepalese Rupee (NPR)",
    "LKR": "Sri Lankan Rupee (LKR)",
    "ZAR": "South African Rand (ZAR)",
}

BG = "#f3f6fb"
NAVY = "#172554"
BLUE = "#2563eb"
MUTED = "#64748b"
WHITE = "#ffffff"
GREEN = "#15803d"


class CurrencyConverterApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Currency Converter | Financial Technology")
        self.root.geometry("760x650")
        self.root.minsize(680, 600)
        self.root.configure(bg=BG)

        self.rate_cache = {}
        self.amount_var = tk.StringVar(value="100")
        self.from_var = tk.StringVar(value="USD")
        self.to_var = tk.StringVar(value="INR")
        self.status_var = tk.StringVar(value="Ready. Connect to the internet to load live rates.")
        self.result_var = tk.StringVar(value="Enter an amount and click Convert")
        self.rate_var = tk.StringVar(value="Exchange rate will appear here.")
        self.last_updated_var = tk.StringVar(value="Rates are fetched when you convert.")

        self._build_ui()
        self.root.after(250, self.refresh_rate_status)

    def _build_ui(self):
        header = tk.Frame(self.root, bg=NAVY, padx=28, pady=22)
        header.pack(fill="x")
        tk.Label(
            header, text="CURRENCY CONVERTER", bg=NAVY, fg=WHITE,
            font=("Segoe UI", 22, "bold")
        ).pack(anchor="w")
        tk.Label(
            header, text="A Python mini project in Financial Technology",
            bg=NAVY, fg="#cbd5e1", font=("Segoe UI", 10)
        ).pack(anchor="w", pady=(5, 0))

        main = tk.Frame(self.root, bg=BG, padx=28, pady=20)
        main.pack(fill="both", expand=True)

        card = tk.Frame(main, bg=WHITE, padx=22, pady=20,
                        highlightbackground="#dbe3ef", highlightthickness=1)
        card.pack(fill="x")

        tk.Label(card, text="Amount", bg=WHITE, fg=NAVY,
                 font=("Segoe UI", 10, "bold")).grid(row=0, column=0, sticky="w", pady=(0, 7))
        tk.Label(card, text="From currency", bg=WHITE, fg=NAVY,
                 font=("Segoe UI", 10, "bold")).grid(row=0, column=1, sticky="w", padx=(16, 0), pady=(0, 7))
        tk.Label(card, text="To currency", bg=WHITE, fg=NAVY,
                 font=("Segoe UI", 10, "bold")).grid(row=0, column=2, sticky="w", padx=(16, 0), pady=(0, 7))

        self.amount_entry = ttk.Entry(card, textvariable=self.amount_var, width=16, font=("Segoe UI", 12))
        self.amount_entry.grid(row=1, column=0, sticky="ew", ipady=7)

        codes = list(CURRENCIES.keys())
        self.from_combo = ttk.Combobox(card, textvariable=self.from_var, values=codes,
                                        state="readonly", width=13, font=("Segoe UI", 11))
        self.from_combo.grid(row=1, column=1, sticky="ew", padx=(16, 0), ipady=6)
        self.to_combo = ttk.Combobox(card, textvariable=self.to_var, values=codes,
                                      state="readonly", width=13, font=("Segoe UI", 11))
        self.to_combo.grid(row=1, column=2, sticky="ew", padx=(16, 0), ipady=6)

        for col in range(3):
            card.grid_columnconfigure(col, weight=1)

        actions = tk.Frame(card, bg=WHITE)
        actions.grid(row=2, column=0, columnspan=3, sticky="ew", pady=(18, 0))
        self.convert_btn = tk.Button(
            actions, text="Convert Currency", command=self.convert,
            bg=BLUE, fg=WHITE, activebackground="#1d4ed8", activeforeground=WHITE,
            font=("Segoe UI", 11, "bold"), relief="flat", padx=18, pady=10, cursor="hand2"
        )
        self.convert_btn.pack(side="left")
        tk.Button(
            actions, text="â‡„ Swap", command=self.swap_currencies,
            bg="#e8eefc", fg=NAVY, activebackground="#dbeafe",
            font=("Segoe UI", 10, "bold"), relief="flat", padx=16, pady=10, cursor="hand2"
        ).pack(side="left", padx=(10, 0))
        tk.Button(
            actions, text="Clear", command=self.clear,
            bg="#f1f5f9", fg=NAVY, activebackground="#e2e8f0",
            font=("Segoe UI", 10), relief="flat", padx=16, pady=10, cursor="hand2"
        ).pack(side="left", padx=(10, 0))

        result_card = tk.Frame(main, bg=WHITE, padx=22, pady=22,
                               highlightbackground="#dbe3ef", highlightthickness=1)
        result_card.pack(fill="x", pady=(16, 0))
        tk.Label(result_card, text="CONVERSION RESULT", bg=WHITE, fg=MUTED,
                 font=("Segoe UI", 9, "bold")).pack(anchor="w")
        tk.Label(result_card, textvariable=self.result_var, bg=WHITE, fg=GREEN,
                 font=("Segoe UI", 20, "bold"), wraplength=650, justify="left").pack(anchor="w", pady=(10, 8))
        tk.Label(result_card, textvariable=self.rate_var, bg=WHITE, fg=NAVY,
                 font=("Segoe UI", 10), wraplength=650, justify="left").pack(anchor="w")
        tk.Label(result_card, textvariable=self.last_updated_var, bg=WHITE, fg=MUTED,
                 font=("Segoe UI", 9), wraplength=650, justify="left").pack(anchor="w", pady=(7, 0))

        status_frame = tk.Frame(main, bg=BG)
        status_frame.pack(fill="x", pady=(13, 0))
        tk.Label(status_frame, textvariable=self.status_var, bg=BG, fg=MUTED,
                 font=("Segoe UI", 9), wraplength=690, justify="left").pack(anchor="w")

        tk.Label(
            main,
            text="Note: Exchange rates fluctuate. This project is for educational use and not financial advice.",
            bg=BG, fg=MUTED, font=("Segoe UI", 8), wraplength=690, justify="left"
        ).pack(anchor="w", pady=(13, 0))

    def refresh_rate_status(self):
        self.status_var.set("Ready. Rates are requested from the online exchange-rate service when Convert is clicked.")

    def convert(self):
        raw_amount = self.amount_var.get().strip().replace(",", "")
        try:
            amount = float(raw_amount)
            if amount <= 0:
                raise ValueError
        except ValueError:
            messagebox.showerror("Invalid amount", "Please enter a valid number greater than zero.")
            self.amount_entry.focus_set()
            return

        source = self.from_var.get()
        target = self.to_var.get()
        if source == target:
            self.result_var.set(f"{amount:,.2f} {target}")
            self.rate_var.set(f"1 {source} = 1 {target}")
            self.last_updated_var.set("No exchange-rate lookup is needed for the same currency.")
            self.status_var.set("Conversion completed.")
            return

        self.convert_btn.config(state="disabled", text="Fetching rate...")
        self.status_var.set(f"Fetching the latest available {source} exchange-rate data...")
        threading.Thread(
            target=self._fetch_and_convert,
            args=(amount, source, target),
            daemon=True
        ).start()

    def _fetch_and_convert(self, amount, source, target):
        try:
            if source not in self.rate_cache:
                request = urllib.request.Request(
                    API_URL.format(source),
                    headers={"User-Agent": "BCA-Currency-Converter/1.0"}
                )
                with urllib.request.urlopen(request, timeout=TIMEOUT_SECONDS) as response:
                    data = json.loads(response.read().decode("utf-8"))
                if data.get("result") != "success" or "rates" not in data:
                    raise RuntimeError(data.get("error-type", "Rate service returned an invalid response"))
                self.rate_cache[source] = {
                    "rates": data["rates"],
                    "time": data.get("time_last_update_utc", "Update time not provided"),
                }

            entry = self.rate_cache[source]
            rate = entry["rates"].get(target)
            if rate is None:
                raise RuntimeError(f"No rate was returned for {target}.")
            converted = amount * float(rate)
            update_time = entry["time"]
            self.root.after(
                0, lambda: self._show_result(amount, source, target, converted, float(rate), update_time)
            )
        except (urllib.error.URLError, TimeoutError, json.JSONDecodeError, RuntimeError, ValueError) as exc:
            self.root.after(0, lambda error=str(exc): self._show_error(error))
        except Exception as exc:
            self.root.after(0, lambda error=str(exc): self._show_error(error))

    def _show_result(self, amount, source, target, converted, rate, update_time):
        self.result_var.set(f"{amount:,.2f} {source}  =  {converted:,.2f} {target}")
        self.rate_var.set(f"Exchange rate: 1 {source} = {rate:,.6f} {target}")
        self.last_updated_var.set(f"Rate data timestamp: {update_time}")
        self.status_var.set("Conversion completed successfully.")
        self.convert_btn.config(state="normal", text="Convert Currency")

    def _show_error(self, error):
        self.status_var.set("Could not retrieve a rate. Check your internet connection and try again.")
        self.convert_btn.config(state="normal", text="Convert Currency")
        messagebox.showerror(
            "Exchange rate unavailable",
            "The application could not fetch exchange-rate data.\n\n"
            f"Details: {error}\n\nPlease check your internet connection and try again."
        )

    def swap_currencies(self):
        source = self.from_var.get()
        target = self.to_var.get()
        self.from_var.set(target)
        self.to_var.set(source)
        self.result_var.set("Currencies swapped. Click Convert to calculate.")
        self.rate_var.set("Exchange rate will appear here.")
        self.last_updated_var.set("Rates are fetched when you convert.")

    def clear(self):
        self.amount_var.set("")
        self.result_var.set("Enter an amount and click Convert")
        self.rate_var.set("Exchange rate will appear here.")
        self.last_updated_var.set("Rates are fetched when you convert.")
        self.status_var.set("Cleared. Enter a new amount.")


def main():
    root = tk.Tk()
    try:
        style = ttk.Style()
        if "clam" in style.theme_names():
            style.theme_use("clam")
        style.configure("TCombobox", padding=5)
        style.configure("TEntry", padding=6)
    except tk.TclError:
        pass
    CurrencyConverterApp(root)
    root.mainloop()


if __name__ == "__main__":
    main()
