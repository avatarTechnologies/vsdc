

#POA WORKFLOW
The basic POA workflow contains several steps:
-SDC needs to sign a transaction then tries to execute SIGN_INVOICE command.
- SAM card returns a byte array containing signature. If no more signatures are allowed and audit procedure is needed a SW_MAX_UNAUDITED_TRANSACTIONS response will be thrown.
- At this point SDC has to start POA sending a START_AUDIT command.
- Once the SAM receives the command, and after making some initial checks, it starts building an object (auditData). This piece of information is returned to the card, although not directly. For privacy and integrity reasons it is encrypted and signed. auditData is composed by the following elements:
The SerialID of the certificate present in the card.
TRANSACTIONS_COUNTER.
LAST_AUDITED_TRANSACTION_COUNTER.
The COUNTERS object.
A random token of 32 Bytes.
- ATAX API contains an endpoint to receive auditData. SDC is responsible to send the request. Server behaviour is explained in the next sections.
- ATAX API answers a HTTP code with status and a JSON payload containing information about the flow.
- At this point If the POA is complete SDC will be able to send VERIFY_AUDIT command to SAM card
- If POA is not complete SDC must wait. There’s an GET ATAX API endpoint (/poa) to get the status. This additional steps are explained later.
- If the VERIFY_AUDIT is complete SAM responses SUCCESS. Otherwise and error is thrown.
