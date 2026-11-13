import os
import math
import re
from collections import Counter
from pypdf import PdfReader, PdfWriter

def calculate_entropy(data: bytes) -> float:
    """Вычисление энтропии Шеннона для поиска скрытых данных (стеганографии)."""
    if not data:
        return 0.0
    entropy = 0.0
    length = len(data)
    counts = Counter(data)
    for count in counts.values():
        p_x = count / length
        entropy -= p_x * math.log2(p_x)
    return round(entropy, 4)

def analyze_stylometry(text: str) -> dict:
    """Базовый стилометрический анализ текста (поиск авторских паттернов)."""
    sentences = [s.strip() for s in re.split(r'[.!?]+', text) if s.strip()]
    words = re.findall(r'\b[а-яА-Яa-zA-Z]+\b', text.lower())
    
    if not words or not sentences:
        return {"avg_sentence_len": 0, "lexical_diversity": 0, "risk": "Низкий"}
    
    avg_len = round(len(words) / len(sentences), 1)
    lexical_diversity = round(len(set(words)) / len(words), 2)
    
    # Эвристика: слишком длинные предложения и высокое лексическое разнообразие легко деанонимизируются
    risk = "Высокий (уникальный авторский стиль)" if (avg_len > 18 and lexical_diversity > 0.7) else "Умеренный"
    return {
        "avg_sentence_len": avg_len,
        "lexical_diversity": lexical_diversity,
        "risk": risk
    }

def sanitize_pdf(input_path: str, output_path: str):
    """Глубокая очистка метаданных PDF и анализ энтропии."""
    with open(input_path, "rb") as f:
        raw_bytes = f.read()
    
    entropy = calculate_entropy(raw_bytes)
    reader = PdfReader(input_path)
    writer = PdfWriter()
    
    extracted_text = ""
    for page in reader.pages:
        writer.add_page(page)
        extracted_text += page.extract_text() or ""
    
    # Удаление всех метаданных
    old_meta = reader.metadata
    writer.add_metadata({"/Producer": "GhostTrace Sanitized Engine v0.1"})
    
    with open(output_path, "wb") as f_out:
        writer.write(f_out)
        
    stylometry = analyze_stylometry(extracted_text)
    return old_meta, entropy, stylometry

def main():
    print("="*55)
    print("   GHOSTTRACE v0.1-Demo | Очистка цифрового следа")
    print("="*55)
    file_path = input("Введите путь к PDF-файлу для анализа и очистки: ").strip('"')
    
    if not os.path.exists(file_path):
        print("[ОШИБКА] Файл не найден.")
        return

    out_path = file_path.replace(".pdf", "_clean.pdf")
    print("\n[1/3] Анализ бинарной структуры и энтропии...")
    old_meta, entropy, stylo = sanitize_pdf(file_path, out_path)
    
    print(f" -> Энтропия файла: {entropy} бит/байт (Норма для PDF: 7.2 - 7.9)")
    if entropy > 7.95:
        print(" [!] ВНИМАНИЕ: Аномально высокая энтропия. Возможна стеганография!")
        
    print("\n[2/3] Обнаруженные и удаленные метаданные:")
    if old_meta:
        for k, v in old_meta.items():
            print(f" - {k}: {v}")
    else:
        print(" - Скрытых тегов не найдено.")
        
    print("\n[3/3] Стилометрический аудит текста:")
    print(f" -> Средняя длина предложения: {stylo['avg_sentence_len']} слов")
    print(f" -> Коэффициент лексического разнообразия: {stylo['lexical_diversity']}")
    print(f" -> Риск деанонимизации автора: {stylo['risk']}")
    
    print("\n" + "="*55)
    print(f"[ГОТОВО] Очищенный файл сохранен: {out_path}")
    print("="*55)

if __name__ == "__main__":
    main()
