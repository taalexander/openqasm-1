Grammar
=======

This is a simplified grammar for Open QASM presented in Extended Backus-Naur
form. The unlisted productions :math:`\langle\mathrm{id}\rangle`,
:math:`\langle\mathrm{real}\rangle` and
:math:`\langle\mathrm{nninteger}\rangle` are defined by the regular
expressions below::

		id        := [a-z][A-Za-z0-9_]*
		real      := ([0-9]+\.[0-9]*|[0-9]*\.[0-9]+)([eE][-+]?[0-9]+)?
		nninteger := -?[1-9]+[0-9]*|0
        ...       := .*

Production rules are defined with plain text and denoted by a colon ``production : rule``.
Keywords are enclosed in quotations ``rule : "keyword"``. Whitespace denotes
concatenation. Alternatives are denoted by ``|`` and may be grouped with
parentheses ``(rule1 | rule2 | ...)``. Optional rules are terminated with a ``?`` (zero or one);
repetition is denoted by ``+`` (one or more) and ``*`` (zero or more). An ellipses
``...`` is used to match any series of token.

.. productionlist::
    program: header generic_statement*
    header: version? include*
    version: "OPENQASM" real ";"
    include: "include" id ( ".inc" | ".qasm" ) ";"
    generic_statement: global_statement
        :| statement
    global_statement: subroutine_declaration
        :| kernel_declaration
        :| quantum_gate_definition
        :| calibration
    statement: expression_statement
        :| declaration_statement
        :| selection_statement
        :| iteration_statement
        :| selection_directive_statement
        :| alias_statement
        :| quantum_statement
        :| timing_box
        :| pragma
        :| comment
    comment: "//" ... "/n" | "/*" ... "*/"
    subroutine_declaration: "def" id subroutine_args return_signature? program_block
    subroutine_args: classical_params | quantum_type_and_id_list
    classical_type_and_id_list: ( declare_type association "," )* declare_type association
    bit_type: "bit" | "creg"
    bit_declaration: bit_type id designator
    declare_type: dependent_type_specifier | independent_type_specifier | bit_type
    dependent_type_declaration: (dependent_type_specifier designator | "fixed" double_designator) id
    dependent_type_specifier: "int" | "uint" | "angle" | "stretch"
    designator: "[" expression "]"
    double_designator: "[" expression "," expression "]"
    independent_type_specifier: "bool" | timing_type
    independent_type_declaration: independent_type_specifier id
    association: ":" id
    return_signature: "->" classical_declaration
    constant_declaration: "const" id "=" expression
    classical_declaration: ( bit_declaration | dependent_type_declaration
        :| independent_type_declaration ) assignment_expression?
    program_block: "{" ( program_block | statement ) "}"
    kernel_declaration: "kernel" id classical_type_and_id_list return_signature? ";"
    alias_statement: "let" id "=" concatenate_expression
    concatenate_expression: "=" ( id range | id "||" id | id "[" expression_list "]"
    expression_statement: expression ";"
        :| "return" expression ";
    expression: expression
        :| numeric_expression
        :| membership_test
        :| expression "[" expression "]"
        :| call "(" expression_list? ")"
        :| expression incrementor
        :| quantum_measurement_instruction
        :| expression_terminator
    numeric_expression: expression binaryop expression
        :| unaryop expression
        :| numeric_terminator
    numeric_terminator: costant | nn_integer
    expression_terminator: real | id | time_terminator
    constant: "pi" | "π" | "tau" | "𝜏" | "euler" | "e"
    expression_list: ( expression "," )* expression
    unaryop: "~" | "!" | builtin_math
    call: expression
        :| builtin_math
        :| cast_operator
    builtin_math: "sin" | "cos" | "tan" | "exp" | "ln" | "sqrt" | "popcount" | "lengthof"
    cast_operator: declare_type
    incrementor: "++" | "--
    pragma: "#pragma {" ... "}"
    declaration_statement: (quantum_declaration | classical_declaration | constant_declaration) ";"
    assignment_expression: assignment_operator expression
    binary_operator: "+" | "-" | "*" | "/"
        :| "<<" | ">>" | "rotr" | "rotl" | "&" | "|" | "^" | "&&" | "||"
        :| ">" | "<" | ">=" | "<=" | "==" | "!="
    assignment_operator: "=" | "+=" | "-=" | "*=" | "/=" |
        :| "<<=" | ">>=" |  "&=" | "|=" | "^=" | "~=" | "->"
    membership_test: id "in" set_declaration
    index_set_declaration: "{" expression_list "}" | range
    range: "[" expression? ":" expression? ( ":" expression )? "]"
    selection_statement: "if (" expression ")" program_block ( "else" program_block )?
    iteration_statement: ( "for" membership_test | "while (" expression ")" ) program_block
    selection_directive_statement: selection_directive ";"
    selection_directive: "break" | "continue" | "exit"
    quantum_gate_definition: "gate" quantum_gate_signature quantum_gate_block
    quantum_gate_signature: id classical_params id_list
    id_list: ( id "," )* id
    index_type: id ( "[" expression "]" )?
    index_list: ( index_type "," )* index_type
    classical_params: ( "(" classical_type_and_id_list? ")" )?
    quantum_gate_block: "{" quantum_gate_call* "}"
    quantum_type_and_id_list: ( quantum_type_and_id "," )* quantum_type_and_id
    quantum_type_and_id: quantum_type designator? association
    quantum_statement: quantum_instruction ";"
    quantum_instruction: quantum_gate_call
        :| quantum_measurement_instruction
        :| barrier_instruction
    quantum_measurement_instruction: "measure" index_list
    quantum_measurement_declaration: quantum_measurement_instruction "->" index_list
        :| index_list "=" quantum_measurement_instruction
    barrier_instruction: "barrier" index_list
    quantum_gate_modifiers: ( "inv" | "pow" "[" nninteger "]" | "ctrl" ) "@"
    quantum_gate_call: quantum_gate_name constant_args index_list | delay_call ";"
    delay_call: "delay" ( "[" id | time_unit "]" )? ( range | index_list )
    quantum_gate_name: "CX" | "U" | "reset" | id
        :| quantum_gate_modifier "@" quantum_gate_name
    quantum_gate_modifier: "inv" | "ctrl"
        :| "pow" nninteger?
    quantum_declaration: quantum_type id designator
    quantum_type: "qubit" | "qreg"
    qubit_id_list: ( qubit_id "," )* qubit_id
    qubit_id: id | physical_qubit_id
    physical_qubit_list: ( physical_qubit_id "," )* physical_qubit_id
    physical_qubit_id: "%" [ "q" ] nninteger
    timing_box: "boxas" id gate_block
        :| "boxto" time_unit gate_block
    timing_type: "length" | "stretch" nninteger?
    time_terminator: time | "stretchinf"
    time: id time_unit? | "lengthof" "(" id ")"
    time_unit: "dt" | "ns" | "us" | "ms" | "s"
    calibration: cal_grammar_declaration | cal_definition
    cal_grammar_declaration: "defcalgrammar" cal_grammar_id ";"
    cal_definition: "defcal" cal_grammar_id? id cal_args physical_qubit_list
        :| return_signature cal_body
    cal_grammar_id: ( "openpulse" | id )*
    cal_args: ( "(" [ id_const_list ] ")" )*
    id_const_list: ( numeric_terminator "," )* numeric_terminator
    cal_body: "{" ... "}"
