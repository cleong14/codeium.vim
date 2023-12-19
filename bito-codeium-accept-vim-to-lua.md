Sure! Here's the Lua code equivalent of the given Vimscript:

```lua
function codeium_Accept()
    local default = vim.g.codeium_tab_fallback or (vim.fn.pumvisible() and "<C-N>") or "\t"
    if not (vim.fn.mode():match('^[iR]') and vim.b._codeium_completions) then
        return default
    end
    local current_completion = codeium_GetCurrentCompletionItem()
    if current_completion == nil then
        return default
    end
    local range = current_completion.range
    local suffix = current_completion.suffix or {}
    local suffix_text = suffix.text or ''
    local delta = suffix.deltaCursorOffset or 0
    local start_offset = range.startOffset or 0
    local end_offset = range.endOffset or 0
    local text = current_completion.completion.text .. suffix_text
    if text == '' then
        return default
    end
    local delete_range = ''
    if end_offset - start_offset > 0 then
        local delete_bytes = end_offset - start_offset
        local delete_chars = vim.fn.strchars(vim.fn.strpart(vim.fn.getline('.'), 0, delete_bytes))
        delete_range = " <Esc>\"_x0\"_d" .. delete_chars .. 'li'
    end
    local insert_text = vim.api.nvim_replace_termcodes("<C-R><C-O>=codeium_CompletionText()<CR>", true, true, true)
    local completion_text = text
    if delta ~= 0 then
        local cursor_text = vim.api.nvim_replace_termcodes(":exe 'go' line2byte(line('.'))+col('.')+(" .. delta .. ")<CR>", true, true, true)
        completion_text = delete_range .. insert_text .. cursor_text
    end
    codeium_server_Request('AcceptCompletion', {metadata = codeium_server_RequestMetadata(), completion_id = current_completion.completion.completionId})
    return completion_text
end
```

Please note that the above code assumes that you have the necessary Lua functions and APIs available in your environment. You may need to adjust or modify the code based on your specific Lua setup.


