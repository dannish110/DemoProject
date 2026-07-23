Yes—that's the right approach. Instead of writing logic specific to one label, create a generic keyword that accepts any locator and always returns the cleaned label text. Then use that keyword everywhere in your framework.

For example:

*** Keywords ***

Get Clean Label
    [Arguments]    ${locator}

    ${text}=    Get Text    ${locator}
    ${text}=    Replace String Using Regexp    ${text}    \u00A0    ${SPACE}
    ${text}=    Replace String Using Regexp    ${text}    \s+    ${SPACE}
    ${text}=    Strip String    ${text}

    [Return]    ${text}

If some labels include child elements like radio buttons (Yes, No) or help text, then Get Text on the container will still return everything. In that case, make the keyword smarter by extracting only the first non-empty line:

Get Clean Label
    [Arguments]    ${locator}

    ${text}=    Get Text    ${locator}
    ${lines}=    Split To Lines    ${text}

    FOR    ${line}    IN    @{lines}
        ${line}=    Strip String    ${line}
        IF    '${line}' != ''
            ${label}=    Set Variable    ${line}
            BREAK
        END
    END

    ${label}=    Replace String Using Regexp    ${label}    \s+    ${SPACE}
    ${label}=    Strip String    ${label}

    [Return]    ${label}

Then every validation becomes:

${actual}=    Get Clean Label    ${locator}
Should Be Equal    ${actual}    ${expected}

An even better long-term solution

If all your form labels follow the same HTML structure, don't pass the container locator. Pass the field locator and have the keyword automatically find its label.

For example:

Validate Label
    [Arguments]    ${field_locator}    ${expected}

    ${label}=    Get Text
    ...    xpath=${field_locator}/preceding-sibling::label

    ${label}=    Replace String Using Regexp    ${label}    \s+    ${SPACE}
    ${label}=    Strip String    ${label}

    Should Be Equal    ${label}    ${expected}

This makes the keyword reusable for every form field without needing separate label locators.

Can you share one sample of your HTML (the <label> and surrounding <div>)? I can suggest an XPath that will work for all labels in your application.
