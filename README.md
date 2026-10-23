bash <<'BASH'
ANALYZER="$PWD/pypi_analyzer.py"

if [[ ! -f "$ANALYZER" ]]; then
    printf 'Không tìm thấy pypi_analyzer.py trong thư mục hiện tại.\n'
    exit 1
fi
if ! command -v python3 >/dev/null 2>&1; then
    printf 'Máy chưa có lệnh python3.\n'
    exit 1
fi

shopt -s nullglob
packages=()
states=()

for directory in ./*/; do
    directory="${directory%/}"
    [[ "${directory##*/}" == "__pycache__" ]] && continue
    packages+=("$directory")
    states+=("CHUA_CHAY")
done

if (( ${#packages[@]} == 0 )); then
    printf 'Không tìm thấy thư mục package cạnh tool.\n'
    exit 1
fi

summary() {
    printf '\n========== TỔNG KẾT ==========\n'
    for i in "${!packages[@]}"; do
        printf '[%-10s] %s\n' \
            "${states[$i]}" "${packages[$i]##*/}"
    done
}

trap summary EXIT
trap 'exit 130' INT
trap 'exit 143' TERM

failed=0
for i in "${!packages[@]}"; do
    package="${packages[$i]}"
    states[$i]="CHUA_XONG"

    printf '\n[%d/%d] Đang xử lý: %s\n' \
        "$((i + 1))" "${#packages[@]}" "${package##*/}"

    if python3 "$ANALYZER" diff "$package" > /dev/null; then
        states[$i]="OK"
    else
        result=$?
        (( result == 130 || result == 143 )) && exit "$result"
        states[$i]="ERROR"
        failed=1
    fi

    printf '[%s] %s\n' "${states[$i]}" "${package##*/}"
done

exit "$failed"
BASH
