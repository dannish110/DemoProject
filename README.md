Get Clean Label
    [Arguments]    ${locator}    ${all_lines}=False

    ${text}=    Get Text    ${locator}
    ${lines}=    Split To Lines    ${text}

    IF    ${all_lines}
        ${label}=    Evaluate    " ".join([l.strip() for l in """${text}""".splitlines() if l.strip()])
    ELSE
        FOR    ${line}    IN    @{lines}
            ${line}=    Strip String    ${line}
            IF    '${line}' != ''
                ${label}=    Set Variable    ${line}
                BREAK
            END
        END
    END

    ${label}=    Evaluate    " ".join("""${label}""".split())

    [Return]    ${label}