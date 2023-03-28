TITLE finalproject.asm
;Program Description: This program uses a variation of Caesar Cipher
;to encrypt or decrypt a user entered phrase using a user entered key
;Author: Mai Nguyen, Anh Tran, Noah Warren
;Creation Date: 11/17/2022

INCLUDE Irvine32.inc
;prototypes are here

displayMenu PROTO,
	uInput:PTR BYTE

pickAProc PROTO,
	userInput:BYTE,
	ptrPhrase:PTR BYTE,
	ptrKey:PTR BYTE,
	ptrPhraseEntered:PTR BYTE,
	ptrKeyEntered:PTR BYTE,
	ptrtempPhrase:PTR BYTE,
	ptrPhraseLength:PTR DWORD,
	ptrKeyLength:PTR DWORD

clearString PROTO,
	clearStr:DWORD

Opt3 PROTO, 
	thePhrase:DWORD, 
	theKey:DWORD, 
	theTempPhrase:DWORD,
	thePhraseLen:DWORD,
	theKeyLen:DWORD

AlphaNums PROTO,
	originalPhrase:DWORD,
	temporaryPhrase:DWORD,
	originalPhraseLen:DWORD

UpperCase PROTO,
	lowerString:DWORD,
	lowerStringLen:DWORD

CopyString PROTO,
	original:DWORD,
	temporary:DWORD,
	originalLen:DWORD

PrintEncrypt PROTO, 
	PrintString: DWORD,
	PrintStringLen: DWORD,

Opt1 PROTO,
	opt1Phrase:DWORD,
	opt1LenPhrase:DWORD

Opt2 PROTO,
	opt2Key:DWORD, 
	opt2LenKey:DWORD

Opt4 PROTO, 
	thePhraseOpt4:DWORD, 
	theKeyOpt4:DWORD, 
	theTempPhraseOpt4:DWORD,
	thePhraseLenOpt4:DWORD,
	theKeyLenOpt4:DWORD	

PrintDecrypt PROTO, 
	PrintStringDecrypt: DWORD,
	PrintStringLenDecrypt: DWORD
	
.data
	userChoiceInvalid BYTE "That is not a valid option please try again.", 0h
	maxStrLen = 151d
	phrase BYTE maxStrLen DUP(0h)
	key BYTE maxStrLen DUP(0h)
	newLine EQU <0ah, 0dh>
	userChoice BYTE 0h
	phraseEntered BYTE 0h
	keyEntered BYTE 0h
	tempPhrase BYTE maxStrLen DUP(0h) 
	phraseLength DWORD 0h
	keyLength DWORD 0h
.code
main PROC

	;INVOKE Opt2, ADDR key, ADDR keyLength
	; display menu will be a loop
	startOfMenu:
		INVOKE displayMenu, ADDR userChoice

		;validate if userChoice is correct
		cmp userChoice, 1d
		jb wrongChoice

		cmp userChoice, 5d
		je EndMenu

		cmp userChoice,5d
		ja wrongChoice

		;if valid, will call pickAProc
		INVOKE pickAProc, userChoice, ADDR phrase, ADDR key, ADDR phraseEntered, 
			ADDR keyEntered, ADDR tempPhrase, ADDR phraseLength, ADDR keyLength
		jmp startOfMenu


		wrongChoice:
			call Crlf
			mov edx, OFFSET userChoiceInvalid
			call WriteString
			call Crlf
			call Crlf
			jmp startOfMenu

	EndMenu:
	exit
main ENDP
;// PROCEDURES

displayMenu PROC,
	uInput:PTR BYTE ;variable storing user choice
;------------------------------------------------------------------
;Displays user menu options 
;Receives: nothing
;Returns: the user input in AL register
;Requires: nothing
;------------------------------------------------------------------
	.data
	MainMenu BYTE "Main Menu", newline,
	"1. Enter a phrase", newline,
	"2. Enter a key", newline,
	"3. Encrypt a phrase", newline,
	"4. Decrypt a phrase", newline,
	"5. Exit", newline,
	"		Please make a selection ==>   ", 0h
	
	.code
	mov EDX, OFFSET MainMenu
	call WriteString
	call ReadDec ; returned in eax
	mov ebx, uInput ; store pointer to userinput in register
	mov [ebx], al


	ret
displayMenu ENDP

pickAProc PROC,
	userInput:BYTE,
	ptrPhrase:PTR BYTE,
	ptrKey:PTR BYTE,
	ptrPhraseEntered:PTR BYTE,
	ptrKeyEntered:PTR BYTE,
	ptrtempPhrase:PTR BYTE,
	ptrPhraseLength:PTR DWORD,
	ptrKeyLength:PTR DWORD
;--------------------------------------------------------------
;calls a procedure from user input
;Receives: user input in userInput variable
;Returns: nothing
;Requires: nothing
;--------------------------------------------------------------
	.data
	callOpt1Prompt BYTE "There is no phrase please pick option 1 first.", 0h
	callOpt2Prompt BYTE "There is no key, please pick option 2 first.", 0h
	phraseQuestionPromptPt1 BYTE "This is the phrase that has been entered : ", 0h
	phraseQuestionPromptPt2 BYTE "Do you want to use this phrase? Enter a number: (1)Yes (2) No =>  ", 0h
	keyQuestionPromptPt1 BYTE "This is the key that has been entered : ", 0h
	keyQuestionPromptPt2 BYTE "Do you want to use this key? Enter a number: (1)Yes (2) No =>  ", 0h
	QuestionPromptWrong BYTE "You have entered an invalid number try again.",0h
	
	.code

	mov edi, ptrPhrase ; contains address to phrase
	mov ebx, ptrKey	   ; contains address to key
	
	cmp userInput, 2d
	jb option1 ; enter a phrase
	
	cmp userInput, 3d
	jb option2 ; enter a key
	
	cmp userInput, 4d
	jb option3 ; encrypt a phrase
	
	cmp userInput, 5d
	jb option4 ; decrypt a phrase

	cmp userInput, 6d
	jb option5 ; exit
	
	;validate if a phrase and key was entered before invoking encrypt and decrypt 
	option1: ; enter phrase
		; invoke opt1
		INVOKE Opt1, ptrPhrase, ptrPhraseLength
		;change variable to 1
		mov eax, 1
		mov ecx, ptrPhraseEntered ;address to t/f for phrase
		mov [ecx], al
		jmp QuitIt
		
	option2: ;enter key
		;invoke opt2
		INVOKE Opt2, ptrKey, ptrKeyLength
		mov eax, 1
		mov ecx, ptrKeyEntered ; address to t/f for key
		mov [ecx], al
		jmp QuitIt

	option3: ;encrypt
		;validation
		mov eax, 1
		mov ecx, ptrPhraseEntered ;address to t/f for phrase
		cmp [ecx], al ; phrase entered check
		jne NeedOpt1

		startOpt3:
			;ask the user if they want to use the phrase
			call crlf
			mov edx, OFFSET phraseQuestionPromptPt1
			call crlf
			call WriteString

			mov edi, ptrPhrase ; contains address to phrase
			mov edx, edi
			call WriteString ;write phrase
			call Crlf

			mov edx, OFFSET phraseQuestionPromptPt2
			call WriteString
			call ReadDec ; user value will be stored in eax

			cmp al, 1h
			jb opt3Wrong
			je keyCheck

			cmp al, 2h
			ja opt3Wrong
			je INeedANewPhrase

			opt3Wrong:
				call crlf
				mov edx, OFFSET QuestionPromptWrong
				call WriteString
				call crlf
				jmp startOpt3
		
			INeedANewPhrase:
				;call opt1 
				INVOKE Opt1, ptrPhrase, ptrPhraseLength
				;change variable to 1
				mov eax, 1
				mov ecx, ptrPhraseEntered ;address to t/f for phrase
				mov [ecx], al

				jmp keyCheck


			keyCheck:
				mov eax, 1
				mov ecx, ptrKeyEntered ; address to t/f for key
				cmp [ecx], al ; key entered check
				jne NeedOpt2

				;ask the user if they want to use the key
				

				keyQuestionCheck:
				call crlf
				mov edx, OFFSET keyQuestionPromptPt1
				call WriteString

				mov ebx, ptrKey	   ; contains address to key
				mov edx, ebx
				call WriteString ;write key
				call Crlf

				mov edx, OFFSET keyQuestionPromptPt2
				call WriteString
				call ReadDec ; user value will be stored in eax

				cmp al, 1h
				jb WrongKeyQuestionInput
				je encryptNow

				cmp al, 2h
				ja WrongKeyQuestionInput
				je INeedANewKey

				WrongKeyQuestionInput:
					call crlf
					mov edx, OFFSET QuestionPromptWrong
					call WriteString
					jmp keyQuestionCheck

				INeedANewKey:
					;call opt 2
					INVOKE Opt2, ptrKey, ptrKeyLength
					mov eax, 1
					mov ecx, ptrKeyEntered ; address to t/f for key
					mov [ecx], al
					jmp encryptNow

				encryptNow:
					
					INVOKE OPT3, ptrPhrase, ptrKey, ptrtempPhrase, ptrPhraseLength, ptrKeyLength
					jmp QuitIt
		
	option4: ;decrypt
		;validation 
		mov eax, 1
		mov ecx, ptrPhraseEntered ;address to t/f for phrase
		cmp [ecx], al ; phrase entered check
		jne NeedOpt1

		startOpt4:
			;ask the user if they want to use the phrase
			call crlf
			mov edx, OFFSET phraseQuestionPromptPt1
			call crlf
			call WriteString

			mov edi, ptrPhrase ; contains address to phrase
			mov edx, edi
			call WriteString ;write phrase
			call Crlf

			mov edx, OFFSET phraseQuestionPromptPt2
			call WriteString
			call ReadDec ; user value will be stored in eax

			cmp al, 1h
			jb opt4Wrong
			je keyCheckOpt4

			cmp al, 2h
			ja opt4Wrong
			je INeedANewPhraseOpt4

			opt4Wrong:
				call crlf
				mov edx, OFFSET QuestionPromptWrong
				call WriteString
				call crlf
				jmp startOpt4
		
			INeedANewPhraseOpt4:
				;call opt1 
				INVOKE Opt1, ptrPhrase, ptrPhraseLength
				;change variable to 1
				mov eax, 1
				mov ecx, ptrPhraseEntered ;address to t/f for phrase
				mov [ecx], al
				jmp keyCheckOpt4


			keyCheckOpt4:
				mov eax, 1
				mov ecx, ptrKeyEntered ; address to t/f for key
				cmp [ecx], al ; key entered check
				jne NeedOpt2

				;ask the user if they want to use the key
				

				keyQuestionCheckOpt4:
				call crlf
				mov edx, OFFSET keyQuestionPromptPt1
				call WriteString

				mov ebx, ptrKey	   ; contains address to key
				mov edx, ebx
				call WriteString ;write key
				call Crlf

				mov edx, OFFSET keyQuestionPromptPt2
				call WriteString
				call ReadDec ; user value will be stored in eax

				cmp al, 1h
				jb WrongKeyQuestionInputOpt4
				je decryptNow

				cmp al, 2h
				ja WrongKeyQuestionInputOpt4
				je INeedANewKeyOpt4

				WrongKeyQuestionInputOpt4:
					call crlf
					mov edx, OFFSET QuestionPromptWrong
					call WriteString
					jmp keyQuestionCheckOpt4

				INeedANewKeyOpt4:
					;call opt 2
					INVOKE Opt2, ptrKey, ptrKeyLength
					mov eax, 1
					mov ecx, ptrKeyEntered ; address to t/f for key
					mov [ecx], al
					jmp decryptNow

				decryptNow:
					INVOKE OPT4, ptrPhrase, ptrKey, ptrtempPhrase, ptrPhraseLength, ptrKeyLength
					jmp QuitIt


	option5:
		jmp QuitIt

	NeedOpt1:
		mov edx, OFFSET callOpt1Prompt
		call Crlf
		call WriteString
		call crlf
		call crlf
		jmp QuitIt

	NeedOpt2:
		mov edx, OFFSET callOpt2Prompt
		call Crlf
		call WriteString
		call crlf
		call crlf
		jmp QuitIt
		
	QuitIt:
	ret
pickAProc ENDP

clearString PROC,
	clearStr:DWORD
;--------------------------------------------------------------
;clear the string of either the key or phrase
;Receives: the address of the phrase/key
;Returns: 
;Requires: nothing
;--------------------------------------------------------------
	mov ecx, 151d
	mov eax, clearStr ;moves address of string into eax
	mov esi, 0 ;indexing
	mov ebx, 0

	loopIt:
		mov BYTE PTR [eax + esi], bl
		inc esi
		loop loopIt

	ret
clearString ENDP
Opt1 PROC,
	opt1Phrase:DWORD,
	opt1LenPhrase:DWORD

;---------------------------------------------------------------------------------
;Requests a phrase from the user
;Receives:the phrase string and the phrase length variable
;Returns: the updated phrase and length 
;Requires: nothing
;---------------------------------------------------------------------------------
	.data
		opt1Prompt BYTE "Please enter a phrase: ", 0h

	.code
		;clear the string before anything
		INVOKE clearString, opt1Phrase

		mov edx, OFFSET opt1Prompt
		call crlf
		call WriteString

		mov ecx, 151d ;max string len 150 + 1 for null terminating byte
		mov edx, opt1Phrase
		call ReadString
		call crlf
		mov esi, opt1LenPhrase  ;store address of len phrase
		mov [esi], eax
		

	ret
Opt1 ENDP

Opt2 PROC,
	opt2Key:DWORD, ;dword because it is an address
	opt2LenKey:DWORD ;dword because it is an address
;--------------------------------------------------------------
;requests a key from the user
;Receives: key string and key length variable
;Returns: key string updated and key length updated
;Requires: nothing
;--------------------------------------------------------------
	.data
		opt2Prompt BYTE "Please enter a key: ", 0h

	.code
		;clear string before anything
		INVOKE clearString, opt2Key

		mov edx, OFFSET opt2Prompt
		call crlf
		call WriteString

		mov ecx, 151d ;max string len 150 + 1 for null terminating byte
		mov edx, opt2Key
		call ReadString
		mov esi, opt2LenKey
		mov [esi], eax
	
	ret
Opt2 ENDP

Opt3 PROC, 
	thePhrase:DWORD, 
	theKey:DWORD, 
	theTempPhrase:DWORD,
	thePhraseLen:DWORD,
	theKeyLen:DWORD
;------------------------------------------------------------------------------------------------------------------
; encrypt phrase
;Receives: the key, phrase, and temp phrase
;Returns:  nothing
;Requires: nothing
;-------------------------------------------------------------------------------------------------------------------
	;// remove all non-alphanumeric from the phrase
	INVOKE AlphaNums, thePhrase, theTempPhrase, thePhraseLen

	;// convert the phrase to uppercases
	INVOKE UpperCase, thePhrase, thePhraseLen
	
	mov esi, 0 ;// use as indexes
	mov eax, 0
	mov ebx, thePhraseLen
	mov ecx, [ebx]
	dec ecx ;// subtract 1 for the null character. Because null character doesn't need to be encrypted.
	mov ebx, 0 

	Encrypting:
		
		;// check if the index of the key is still in bound
		mov edx, theKeyLen ;// copy the value of the key length
		mov eax, [edx]
		cmp ebx, eax ;// check if the index of the key is out of bound or not
		je ResetKey ;// if it is out of bound

		mov edx, 0d ;// ignore if it's in bound
		mov eax, 0d
		jmp Cont

		ResetKey:
		mov EBX, 0h ;// go back to the beginning of the key

		Cont:
		mov dl, [EDI + ESI] ;// copy the character of phrase to dl
		
		push EDI ;// store the address of the phrase on stack
		mov EDI, theKey ;// copy the address of the key to EDI
		;//check if the character is a letter or a number
		;// 0-9 is 30h to 39h
		;// A-Z is 41h to 5Ah
		
		cmp dl, 39h ;// bound of numbers
		ja IsaLetter
		jbe IsaNumber
		
		IsaLetter: 
			mov al, [EDI + EBX] ;// move the character of the key to al with zero extension
			
			push ebx
			mov ebx, 1AH
			div bl ;// remainder is saved in AH
			pop ebx

			sub dl, ah ;// subtract the remainder from the phrase character
			cmp dl, 41h;// check if it is out of bound or not
			jb OutOfBoundLetter ;// if the result is out of bound
			jmp NextChar ;// if the result is in the bound
		
		OutOfBoundLetter:
			add dl, 1AH ;// add 1AH to go back to the bound
			jmp NextChar
		
		IsaNumber:
			mov al, [EDI + EBX] ;// move the character of the key to al with zero extension

			push ebx 
			mov ebx, 0AH 
			div bl;// remainder is saved in AH
			pop ebx

			sub dl, ah ;// subtract the remainder from the phrase character
			cmp dl, 30h;// check if it is out of bound or not
			jb OutOfBoundNumber ;// if the result is out of bound
			jmp NextChar ;// if the result is in the bound
			
		OutOfBoundNumber:
			add dl, 0AH ;// add 0AH to go back to the bound
			jmp NextChar
		
		NextChar: 
			pop EDI ;// return the address of the phrase to EDI
			mov [EDI + ESI], dl ;// copy the encrypted character back to the phrase
			inc ESI ;// move to the next character of the phrase
			inc EBX ;// move to the next character of the key
			loop Encrypting 
	
	;// call a proc to print out the encrypted string
	INVOKE PrintEncrypt, thePhrase, thePhraseLen
	ret
Opt3 ENDP

PrintEncrypt PROC, 
	PrintString: DWORD,
	PrintStringLen: DWORD
;---------------------------------------------------------------------------------
;Print the string with the required format
;Receives: phrase and the length of the phrase
;Returns: nothing
;Requires: nothing
;---------------------------------------------------------------------------------
	.data
	printprompt BYTE "The encrypted string: ",0d

	.code

	call clrscr ;// clear the console window to use gotoxy
	mov edx, OFFSET printprompt
	call writestring
	mov edx, 0
	mov edi, PrintString ;// point to the address of the phrase
	mov ecx, 0
	mov ebx, PrintStringLen
	mov ecx, [ebx]
	mov eax, 0
	mov esi, 0 ;// use for index of the string
	mov ebx, 0 ;// character counter

	PrintStringLoop:
		mov eax, 0 ;// clear eax after every iteration
		cmp ebx, 7d ;// if 7 characters are printed
		je PrintSpace
		jmp Contprint

		PrintSpace:
			add dl, 3d ;// add 3 spaces to the line
			mov ebx, 0;// reset character counter

		Contprint:
			mov al, [EDI + ESI] ;// copy each character in the phrase
			mov dh, 1 ;// default setting for row
			
			call gotoxy
			call WriteChar ;// print out a character
			
			inc dl ;// next column
			inc ESI ;// next character in the string
			inc ebx ;// character counter
			loop PrintStringLoop
	
	call crlf
	call waitmsg
	call crlf
	ret
PrintEncrypt ENDP

AlphaNums PROC,
	originalPhrase:DWORD,
	temporaryPhrase:DWORD,
	originalPhraseLen:DWORD
;---------------------------------------------------------------------------------
;remove all non alphanumeric from the phrase
;Receives: phrase and temp phrase
;Returns: the phrase changed to be alphanumeric only
;Requires: nothing
;---------------------------------------------------------------------------------
	;clear temp phrase
	INVOKE clearString, temporaryPhrase

	mov edi, originalPhrase
	mov ebx, temporaryPhrase
	mov esi, 0 
	
	mov edx, originalPhraseLen
	mov ecx, [edx]
	mov edx, 0
	mov eax, 0

	removeIt:
		mov al, [EDI + ESI]

		cmp al, 30h
		jb continueOn
		cmp al, 39h
		jbe addstring

		;check if uppercase char (41h-5Ah)
		cmp al, 41h
		jb continueOn
		cmp al, 5Ah
		jbe addstring

		;check if lowercase char (61h-7Ah)
		cmp al, 61h
		jb continueOn
		cmp al, 7Ah
		jbe addstring

		;anything else larger than 7Ah will continue on
		jmp continueOn

		addString:
			mov [EBX + EDX], al
			inc EDX
		continueOn: ;do nothing just move on to next index
			inc esi
		loop removeIt

		inc edx; add 1 for the null terminator
		mov eax, originalPhraseLen
		mov [eax], edx 
		
	;copy temp phrase over to original phrase
	INVOKE CopyString, originalPhrase, temporaryPhrase, originalPhraseLen
	
	ret
AlphaNums ENDP

CopyString PROC,
	original:DWORD,
	temporary:DWORD,
	originalLen:DWORD
;------------------------------------------------------------------------------------
;copies the string from tempstring to actual string
;Receives: the phrase string, the temp phrase string, and the length of the phrase
;Returns: updated string in edx that was copied from tempstring.
;Requires: nothing
;-------------------------------------------------------------------------------------

	;clear original string
	INVOKE clearString, original

	mov edi, original; address of the phrase
	mov ebx, temporary ; address of the tempphrase

	mov edx, originalLen; address of the phrase len
	mov ecx, [edx]; move actual value of phrase len

	; indexer
	mov esi, 0

	copyStringLoop:
		mov al, BYTE PTR[ebx + esi]
		mov BYTE PTR [edi + esi], al
		inc esi
		loop copyStringLoop
	

	ret
CopyString ENDP

UpperCase PROC,
	lowerString:DWORD,
	lowerStringLen:DWORD
;---------------------------------------------------------------------------------
;changes all the lowercase characters to uppercase
;Receives: the phrase string, the phraselength string
;Returns: the phrase string with all lowercase characters changed to uppercase
;Requires: nothing
;---------------------------------------------------------------------------------
	mov ebx, lowerString
	mov esi, 0 ; indexer
	mov edx, lowerStringLen; move address of length
	mov ecx, [edx]; move actual value of length into ecx
	mov eax, 0

	upperCaseLoop:
		mov al, byte ptr [ebx + esi]
		
		;see if it is a lower case chara
		cmp al, 61h
		jb cont

		cmp bl, 7Ah
		ja cont
		and byte ptr [ebx+esi], 0DFh ; converts to uppercase


		;//will go to next loop
		cont:
			inc esi
			loop upperCaseLoop
	
	ret

UpperCase ENDP

Opt4 PROC, 
	thePhraseOpt4:DWORD, 
	theKeyOpt4:DWORD, 
	theTempPhraseOpt4:DWORD,
	thePhraseLenOpt4:DWORD,
	theKeyLenOpt4:DWORD
;------------------------------------------------------------------------------------------------------------------
;decrypt phrase
;Receives: the key, phrase,temp phrase, the length of the phrase and length of the key
;Returns:  decrypted phrase
;Requires: nothing
;-------------------------------------------------------------------------------------------------------------------
	

	;// remove all non-alphanumeric from the phrase
	INVOKE AlphaNums, thePhraseOpt4, theTempPhraseOpt4, thePhraseLenOpt4
	;// convert the phrase to uppercases
	INVOKE UpperCase, thePhraseOpt4, thePhraseLenOpt4
	
	mov esi, 0 ;// use as indexes
	mov eax, 0
	mov ebx, thePhraseLenOpt4
	mov ecx, [ebx]
	dec ecx ;// subtract 1 for the null character. Because null character doesn't need to be decrypted.
	mov ebx, 0 

	Decrypting:
		
		;// check if the index of the key is still in bound
		mov edx, theKeyLenOpt4 ;// copy the value of the key length
		mov eax, [edx]
		cmp ebx, eax ;// check if the index of key is out of bound or not
		je ResetKeyOpt4 ;// if it is out of bound

		mov edx, 0d ;// ignore if it's in bound
		mov eax, 0d
		jmp ContOpt4

		ResetKeyOpt4:
		mov EBX, 0h ;// go back to the beginning of the key

		ContOpt4:
		mov dl, [EDI + ESI] ;// copy the character of phrase to dl
		
		mov EDI, theKeyOpt4 ;// copy the address of the key to EDI
		;//check if the character is a letter or a number
		;// 0-9 is 30h to 39h
		;// A-Z is 41h to 5Ah
		
		cmp dl, 39h ;// bound of numbers
		ja IsaLetterOpt4
		jbe IsaNumberOpt4
		
		IsaLetterOpt4: 
			mov al, [EDI + EBX] ;// move the character of the key to al with zero extension
			
			push ebx
			mov ebx, 1AH
			div bl ;// remainder is saved in AH
			pop ebx

			add dl, ah ;// subtract the remainder from the phrase character
			cmp dl, 5Ah;// check if it is out of bound or not
			ja OutOfBoundLetterOpt4 ;// if the result is out of bound
			jmp NextCharOpt4 ;// if the result is in the bound
		
		OutOfBoundLetterOpt4:
			sub dl, 1AH ;// add 1AH to go back to the bound
			jmp NextCharOpt4
		
		IsaNumberOpt4:
			mov al, [EDI + EBX] ;// move the character of the key to al with zero extension

			push ebx 
			mov ebx, 0AH 
			div bl;// remainder is saved in AH
			pop ebx

			add dl, ah ;// subtract the remainder from the phrase character
			cmp dl, 39h;// check if it is out of bound or not
			ja OutOfBoundNumberOpt4 ;// if the result is out of bound
			jmp NextCharOpt4 ;// if the result is in the bound
			
		OutOfBoundNumberOpt4:
			sub dl, 0AH ;// add 0AH to go back to the bound
			jmp NextCharOpt4
		
		NextCharOpt4: 
			mov edi, thePhraseOpt4 ;// return the address of the phrase to EDI
			mov [EDI + ESI], dl ;// copy the encrypted character back to the phrase
			inc ESI ;// move to the next character of the phrase
			inc EBX ;// move to the next character of the key
			loop Decrypting 
	
	;// call a proc to print out the encrypted string
	INVOKE PrintDecrypt, thePhraseOpt4, thePhraseLenOpt4
	ret
Opt4 ENDP

PrintDecrypt PROC, 
	PrintStringDecrypt: DWORD,
	PrintStringLenDecrypt: DWORD
;---------------------------------------------------------------------------------
;Print the string with the required format
;Receives: phrase and the length of the phrase
;Returns: nothing
;Requires: nothing
;---------------------------------------------------------------------------------
	.data
	printpromptDecrypt BYTE "The decrypted string: ",0d
	.code

	call clrscr ;// clear the console window to use gotoxy
	mov edx, OFFSET printpromptDecrypt
	call writestring
	mov edx, 0
	mov edi, PrintStringDecrypt ;// point to the address of the phrase
	mov ecx, 0
	mov ebx, PrintStringLenDecrypt
	mov ecx, [ebx]
	mov eax, 0
	mov esi, 0 ;// use for index of the string
	mov ebx, 0 ;// character counter

	PrintStringLoopDecrypt:
		mov eax, 0 ;// clear eax after every iteration
		cmp ebx, 7d ;// if 7 characters are printed
		je PrintSpaceDecrypt
		jmp ContprintDecrypt

		PrintSpaceDecrypt:
			add dl, 3d ;// add 3 spaces to the line
			mov ebx, 0;// reset character counter

		ContprintDecrypt:
			mov al, [EDI + ESI] ;// copy each character in the phrase
			mov dh, 1 ;// default setting for row
			
			call gotoxy
			call WriteChar ;// print out a character
			
			inc dl ;// next column
			inc ESI ;// next character in the string
			inc ebx ;// character counter
			loop PrintStringLoopDecrypt
	
	call crlf
	call waitmsg
	call crlf
	ret
PrintDecrypt ENDP


END main
