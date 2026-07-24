Get Multi Line Label
    [Arguments]    ${locator}

    ${element}=    Get WebElement    ${locator}
    ${label}=    Execute JavaScript
    ...    return arguments[0].childNodes[0].textContent.trim() + " " +
    ...           arguments[0].childNodes[2].textContent.trim();
    ...    ${element}

    ${label}=    Evaluate    " ".join("""${label}""".split())
    [Return]    ${label}