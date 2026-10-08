"""
Delta (Δ) Connection Calculator
-------------------------------
A menu-driven program for engineering students to analyse a three-phase
delta-connected system.

Key relations (balanced delta connection):
    Line voltage     VL = Vph
    Line current     IL = sqrt(3) * Iph
    Phase current    Iph = IL / sqrt(3) = Vph / Zph

    Apparent power   S = 3 * Vph * Iph = sqrt(3) * VL * IL     (VA)
    Active power    P = sqrt(3) * VL * IL * cos(phi)           (W)
    Reactive power  Q = sqrt(3) * VL * IL * sin(phi)           (VAR)

Unbalanced delta (phasor analysis):
    Iab = Vab / Zab,  Ibc = Vbc / Zbc,  Ica = Vca / Zca
    Ia = Iab - Ica,   Ib = Ibc - Iab,   Ic = Ica - Ibc
    (Ia + Ib + Ic = 0 always, as there is no neutral)

Star-delta conversion (balanced):
    Z_delta = 3 * Z_star        Z_star = Z_delta / 3

For the same load impedance and line voltage, a delta connection takes
three times the power (and three times the line current) of a star connection.
"""

import cmath
import math

SQRT3 = math.sqrt(3)


# ---------------------------------------------------------------- input helpers
def get_positive_float(prompt):
    """Keep asking until the user enters a valid positive number."""
    while True:
        try:
            value = float(input(prompt))
            if value <= 0:
                print("  Please enter a value greater than zero.")
                continue
            return value
        except ValueError:
            print("  Invalid input. Please enter a number.")


def get_float(prompt):
    """Ask for any number (zero and negative values allowed)."""
    while True:
        try:
            return float(input(prompt))
        except ValueError:
            print("  Invalid input. Please enter a number.")


def get_impedance(label):
    """Ask for resistance and reactance of one phase and return complex Z.
    Reactance: positive for inductive, negative for capacitive."""
    r = get_positive_float(f"  Phase {label}: resistance R (ohm): ")
    x = get_float(f"  Phase {label}: reactance X (ohm, - for capacitive): ")
    return complex(r, x)


# ------------------------------------------------------------------ core maths
def line_current(i_ph):
    """IL = sqrt(3) * Iph"""
    return SQRT3 * i_ph


def phase_current(i_l):
    """Iph = IL / sqrt(3)"""
    return i_l / SQRT3


def balanced_powers(v_l, i_l, pf):
    """Return S, P, Q for a balanced three-phase load."""
    s = SQRT3 * v_l * i_l
    p = s * pf
    q = s * math.sqrt(max(1 - pf ** 2, 0))
    return s, p, q


def delta_to_star(z_delta):
    """Z_star = Z_delta / 3"""
    return z_delta / 3


def star_to_delta(z_star):
    """Z_delta = 3 * Z_star"""
    return 3 * z_star


# ------------------------------------------------------------------- display
def show_polar(name, value, unit):
    """Print a complex quantity in polar form."""
    mag, ang = cmath.polar(value)
    print(f"  {name} = {mag:.3f} < {math.degrees(ang):.2f} deg {unit}")


def line_from_phase():
    v_ph = get_positive_float("Phase voltage Vph (V): ")
    i_ph = get_positive_float("Phase current Iph (A): ")
    print("\n  ----- Results -----")
    print(f"  Line voltage VL = Vph = {v_ph:.3f} V")
    print(f"  Line current IL = sqrt(3) x {i_ph:.2f} = {line_current(i_ph):.3f} A")


def phase_from_line():
    v_l = get_positive_float("Line voltage VL (V): ")
    i_l = get_positive_float("Line current IL (A): ")
    print("\n  ----- Results -----")
    print(f"  Phase voltage Vph = VL = {v_l:.3f} V")
    print(f"  Phase current Iph = {i_l:.2f} / sqrt(3) = {phase_current(i_l):.3f} A")


def balanced_load():
    v_l = get_positive_float("Line voltage VL (V): ")
    r = get_positive_float("Resistance per phase R (ohm): ")
    x = get_float("Reactance per phase X (ohm, - for capacitive): ")

    z = complex(r, x)
    i_ph = v_l / abs(z)
    i_l = line_current(i_ph)
    pf = r / abs(z)
    s, p, q = balanced_powers(v_l, i_l, pf)

    print("\n  ----- Balanced Delta Load -----")
    print(f"  Phase voltage Vph       = {v_l:.3f} V")
    print(f"  Impedance per phase Zph = {abs(z):.3f} ohm")
    print(f"  Phase current Iph       = {i_ph:.4f} A")
    print(f"  Line current IL         = {i_l:.4f} A")
    print(f"  Power factor            = {pf:.4f} {'lagging' if x > 0 else 'leading' if x < 0 else '(unity)'}")
    print(f"  Apparent power S        = {s:,.2f} VA")
    print(f"  Active power P          = {p:,.2f} W")
    print(f"  Reactive power Q        = {q:,.2f} VAR")


def unbalanced_load():
    v_l = get_positive_float("Line voltage VL (V): ")
    print("\n  Enter the impedance of each phase of the delta:")
    zab, zbc, zca = get_impedance("AB"), get_impedance("BC"), get_impedance("CA")

    # Line voltages (ABC sequence, Vab as reference)
    vab = cmath.rect(v_l, 0)
    vbc = cmath.rect(v_l, math.radians(-120))
    vca = cmath.rect(v_l, math.radians(120))

    iab, ibc, ica = vab / zab, vbc / zbc, vca / zca
    ia, ib, ic = iab - ica, ibc - iab, ica - ibc
    total_p = (abs(iab) ** 2 * zab.real) + (abs(ibc) ** 2 * zbc.real) + (abs(ica) ** 2 * zca.real)

    print("\n  ----- Unbalanced Delta Load -----")
    show_polar("Phase current Iab", iab, "A")
    show_polar("Phase current Ibc", ibc, "A")
    show_polar("Phase current Ica", ica, "A")
    show_polar("Line current  Ia ", ia, "A")
    show_polar("Line current  Ib ", ib, "A")
    show_polar("Line current  Ic ", ic, "A")
    print(f"  Total active power P = {total_p:,.2f} W")


def compare_star_delta():
    v_l = get_positive_float("Line voltage VL (V): ")
    r = get_positive_float("Resistance per phase R (ohm): ")
    x = get_float("Reactance per phase X (ohm, - for capacitive): ")

    z = complex(r, x)
    pf = r / abs(z)

    i_star = (v_l / SQRT3) / abs(z)
    i_delta = SQRT3 * (v_l / abs(z))
    p_star = balanced_powers(v_l, i_star, pf)[1]
    p_delta = balanced_powers(v_l, i_delta, pf)[1]

    print("\n  ----- Same load connected in Star and Delta -----")
    print(f"  {'':22}{'Star':>14}{'Delta':>14}")
    print(f"  {'Line current (A)':22}{i_star:>14.3f}{i_delta:>14.3f}")
    print(f"  {'Active power (W)':22}{p_star:>14,.1f}{p_delta:>14,.1f}")
    print(f"  Delta takes {i_delta / i_star:.1f} times the line current and power of star.")


def conversion():
    print("  1. Delta to star")
    print("  2. Star to delta")
    while True:
        pick = input("  Enter 1 or 2: ").strip()
        if pick in ("1", "2"):
            break
        print("  Please enter 1 or 2.")

    z = get_impedance("(balanced)")
    if pick == "1":
        zs = delta_to_star(z)
        print(f"\n  Z_star  = {zs.real:.3f} + j{zs.imag:.3f} ohm per phase")
    else:
        zd = star_to_delta(z)
        print(f"\n  Z_delta = {zd.real:.3f} + j{zd.imag:.3f} ohm per phase")


def menu():
    print("\n" + "=" * 52)
    print("        DELTA CONNECTION CALCULATOR")
    print("=" * 52)
    print(" 1. Line values from phase values")
    print(" 2. Phase values from line values")
    print(" 3. Balanced delta load (current and power)")
    print(" 4. Unbalanced delta load (phasor currents)")
    print(" 5. Compare star and delta for the same load")
    print(" 6. Star-delta impedance conversion")
    print(" 0. Exit")
    print("-" * 52)


def main():
    while True:
        menu()
        choice = input("Enter your choice: ").strip()

        if choice == "1":
            line_from_phase()
        elif choice == "2":
            phase_from_line()
        elif choice == "3":
            balanced_load()
        elif choice == "4":
            unbalanced_load()
        elif choice == "5":
            compare_star_delta()
        elif choice == "6":
            conversion()
        elif choice == "0":
            print("\nThank you for using the calculator. Goodbye!")
            break
        else:
            print("  Invalid choice. Please select from the menu.")


if __name__ == "__main__":
    main()
