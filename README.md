# shiwangi

# textfile.txt
START 100                                                                                                                   
A DC 10
MOVER AREG, B
MOVEM BREG, ='1'
ADD AREG, ='2'
SUB BREG, ='1'
B DC 20
ORIGIN 300
LTORG
MOVER AREG, NUM
MOVER CREG, LOOP
ADD BREG, ='1'
NUM DS 5
LOOP DC 10
END



# assembly.ipynb
import os

LC = 0  
 # mnemonic -> (opcode, class, operand_count)
mnemonics = {
    'STOP': ('00', 'IS', 0), 'ADD': ('01', 'IS', 2), 'SUB': ('02', 'IS', 2), 'MUL': ('03', 'IS', 2),
    'MOVER': ('04', 'IS', 2), 'MOVEM': ('05', 'IS', 2), 'COMP': ('06', 'IS', 2), 'BC': ('07', 'IS', 2),
    'DIV': ('08', 'IS', 2), 'READ': ('09', 'IS', 1), 'PRINT': ('10', 'IS', 1),
    'START': ('01', 'AD', 1), 'END': ('02', 'AD', 0), 'ORIGIN': ('03', 'AD', 1),
    'EQU': ('04', 'AD', 2), 'LTORG': ('05', 'AD', 0),
    'DS': ('01', 'DL', 1), 'DC': ('02', 'DL', 1)
}

# Input and output files
file = open("textfile.txt", "r")   # Input Assembly Program
ifp = open("inter_code.txt", "w")    # Intermediate Code Output

# Register codes
REG = {'AREG': 1, 'BREG': 2, 'CREG': 3, 'DREG': 4}

# Tables
symtab = {}        
pooltab = []         
literal_list = []    
symindex = 0        




def print_symbol_table():
    print("\nSymbol Table:")
    for sym, val in symtab.items():
        print(f"{sym}\tAddress: {val[0]}\tIndex: {val[1]}")


def print_literal_table():
    print("\nLiteral Table:")
    for i, lit in enumerate(literal_list):
        print(f"{i+1}. Literal: {lit['name']}\tAddress: {lit['address']}")


def print_pool_table():
    print("\nPool Table:")
    print(pooltab)



def END():
    
    global LC
    ifp.write("\t(AD,02)\n")
    assign_literals()
    save_tables()
    print("\n--- Assembler Output Tables ---")
    print_symbol_table()
    print_literal_table()
    print_pool_table()
    print("\nPass 1 Completed Successfully ✅")


def LTORG():
    
    global LC
    ifp.write("\t(AD,05)\n")
    assign_literals()


def ORIGIN(addr):
    
    global LC
    LC = int(addr)
    ifp.write(f"\t(AD,03)\t(C,{addr})\n")


def DS(size):
    
    global LC
    ifp.write(f"\t(DL,01)\t(C,{size})\n")
    LC += int(size)


def DC(value):
    
    global LC
    ifp.write(f"\t(DL,02)\t(C,{value})\n")
    LC += 1




def add_literal(literal):
    
    for lit in literal_list:
        if lit['name'] == literal:
            return
    literal_list.append({'name': literal, 'address': None})


def assign_literals():
    
    global LC
    pooltab.append(len(literal_list))  # record the count (start index of pool)
    for lit in literal_list:
        if lit['address'] is None:
            lit['address'] = LC
            ifp.write(f"\t(DL,02)\t(C,{lit['name'][2:-1]})\n")
            LC += 1




def process_instruction(mnemonic, operands):
    #Processes Imperative Statements like MOVER, ADD, etc
    global LC, symindex

    opcode, mclass, operand_count = mnemonics[mnemonic]
    ifp.write(f"\t({mclass},{opcode})\t")

    for op in operands:
        op = op.replace(",", "")
        if op in REG:
            ifp.write(f"(RG,{REG[op]})")
        elif "=" in op:
            add_literal(op)
            index = literal_list.index(next(l for l in literal_list if l['name'] == op))
            ifp.write(f"(L,{index+1})")
        else:
            if op not in symtab:
                symtab[op] = ("**", symindex)
                symindex += 1
            idx = symtab[op][1]
            ifp.write(f"(S,{idx})")
    ifp.write("\n")
    LC += 1



def detect_mn(words):
    
    global LC

    if words[0] == "START":
        LC = int(words[1])
        ifp.write(f"\t(AD,01)\t(C,{LC})\n")

    elif words[0] == "END":
        END()

    elif words[0] == "LTORG":
        LTORG()

    elif words[0] == "ORIGIN":
        ORIGIN(words[1])

    elif words[0] == "DS":
        DS(words[1])

    elif words[0] == "DC":
        DC(words[1])

    elif words[0] in mnemonics.keys():
        process_instruction(words[0], words[1:])




def save_tables():
    
    # Symbol Table
    with open("SymbolTable.txt", "w") as symf:
        symf.write("Symbol\tAddress\tIndex\n")
        for sym, val in symtab.items():
            symf.write(f"{sym}\t{val[0]}\t{val[1]}\n")

    # Literal Table
    with open("LiteralTable.txt", "w") as litf:
        litf.write("Index\tLiteral\tAddress\n")
        for i, lit in enumerate(literal_list):
            litf.write(f"{i+1}\t{lit['name']}\t{lit['address']}\n")

    # Pool Table
    with open("PoolTable.txt", "w") as poolf:
        poolf.write("PoolTable Entries\n")
        for entry in pooltab:
            poolf.write(f"{entry}\n")



#Main Logic
for line in file:
    line = line.strip()
    if not line:
        continue

    words = line.split()
    print(f"\nProcessing line: {line}")

    # Label Handling
    if words[0] not in mnemonics.keys():
        label = words[0]
        if label not in symtab:
            symtab[label] = (LC, symindex)
            symindex += 1
        else:
            old = symtab[label]
            if old[0] == "**":
                symtab[label] = (LC, old[1])
        detect_mn(words[1:])
    else:
        detect_mn(words)

    if words[0] != "START":
        ifp.write(str(LC) + "\n")


file.close()
ifp.close()


# Priority Scheduling (Non-Preemptive) with Different Arrival Times

proc = [[1, 3, 3, 1], [2, 5, 4, 2], [3, 1, 1, 3], [4, 7, 7, 4], [5, 4, 8, 5]]
proc.sort(key=lambda x: (x[0], x[2]))  # Sort by arrival, then priority
n = len(proc)

def priority_scheduling(p):
    wt, tat, st, ct = [0]*n, [0]*n, [0]*n, [0]*n
    for i in range(1, n):
        st[i] = max(p[i][0], st[i-1] + p[i-1][1])
        wt[i] = max(0, st[i] - p[i][0])
    for i in range(n):
        ct[i] = st[i] + p[i][1]
        tat[i] = p[i][1] + wt[i]

    print("P\tAT\tBT\tPR\tST\tCT\tTAT\tWT")
    for i in range(n):
        print(f"P{p[i][3]}\t{p[i][0]}\t{p[i][1]}\t{p[i][2]}\t{st[i]}\t{ct[i]}\t{tat[i]}\t{wt[i]}")

    print("\nAvg WT = {:.2f}".format(sum(wt)/n))
    print("Avg TAT = {:.2f}".format(sum(tat)/n))

priority_scheduling(proc)
