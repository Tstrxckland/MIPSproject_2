# MIPSproject_2
    .data
strPrompt:  .asciiz "Enter a string: "
resultMsg:  .asciiz "Result: "
naMsg:      .asciiz "N/A"
separator:  .asciiz ";"

    .text
    .globl main

main:
    # Prompt user for input
    li $v0, 4
    la $a0, strPrompt
    syscall

    # Read user input
    li $v0, 8
    la $a0, buffer
    li $a1, 1000
    syscall

    # Pass address of input buffer to subprogram
    move $a0, $t0
    jal processSubstring

    # Print result
    li $v0, 4
    la $a0, resultMsg
    syscall

    # Check return value
    bge $v0, 4294967295, printNA
    li $v0, 1
    syscall
    j endProgram

printNA:
    li $v0, 4
    la $a0, naMsg
    syscall

endProgram:
    # Exit
    li $v0, 10
    syscall

processSubstring:
    # Initialize G and H
    li $t1, 0 # G (sum of even index digits)
    li $t2, 0 # H (sum of odd index digits)

    # Loop through the string
    li $t3, 0 # Index of current character
loop:
    lb $t4, 0($a0)      # Load character into $t4
    beqz $t4, done      # If null terminator, end loop
    addi $t3, $t3, 1    # Increment index

    # Check if character is a valid digit
    li $t5, '0'
    sub $t6, $t4, $t5
    bltz $t6, notDigit   # If character is not a digit, ignore
    li $t5, '9'
    slt $t6, $t4, $t5
    bgez $t6, checkLowercase
    li $t5, 'A'
    sub $t6, $t4, $t5
    bltz $t6, notDigit   # If character is not a digit, ignore
    li $t5, 'Z'
    slt $t6, $t4, $t5
    bgez $t6, checkUppercase

notDigit:
    addi $a0, $a0, 1    # Move to next character
    j loop

checkLowercase:
    li $t5, 'a'
    sub $t6, $t4, $t5
    bltz $t6, notLowerCase # If character is not lowercase, ignore
    bgez $t6, calculateValue

notLowerCase:
    li $t5, 'z'
    sub $t6, $t4, $t5
    bltz $t6, notDigit   # If character is not a digit, ignore
    bgez $t6, calculateValue

checkUppercase:
    li $t5, 'A'
    sub $t6, $t4, $t5
    bltz $t6, notUppercase # If character is not uppercase, ignore
    bgez $t6, calculateValue

notUppercase:
    li $t5, 'Z'
    sub $t6, $t4, $t5
    bltz $t6, notDigit   # If character is not a digit, ignore
    bgez $t6, calculateValue

calculateValue:
    # Calculate digit value
    sub $t7, $t4, $t5
    li $t8, 1
    bgt $t7, $zero, evenIndex
    sub $t7, $t3, $t8   # Convert index to 0-based
    b oddIndex

evenIndex:
    add $t1, $t1, $t7   # Add to G
    j incrementIndex

oddIndex:
    add $t2, $t2, $t7   # Add to H
    j incrementIndex

incrementIndex:
    addi $a0, $a0, 1    # Move to next character
    j loop

done:
    # Calculate G - H
    sub $v0, $t1, $t2

    # Return
    jr $ra

    .data
buffer: .space 1000
