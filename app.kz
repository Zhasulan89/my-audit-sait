import os
import streamlit as st
from crewai import Agent, Task, Crew, Process
from crewai_tools import FileReadTool

st.set_page_config(page_title="ИИ-Аудит договоров", page_icon="🛡️", layout="wide")
st.title("🛡️ Виртуальный офис: Аудит договоров и расчет рентабельности")
st.markdown("### Специализированный ИИ-анализ контрактов для ИП на ОУР (Защита интересов Исполнителя)")

with st.sidebar:
    st.header("⚙️ Настройки системы")
    api_key = st.text_input("Введите ваш OpenAI API Key:", type="password")
    st.subheader("📊 Параметры для расчета:")
    fot_percent = st.slider("Фонд оплаты труда (% от договора)", min_value=10, max_value=90, value=50)
    employees_count = st.number_input("Количество сотрудников в ИП", min_value=0, max_value=100, value=2)
    st.info("Расчет налогов производится по ставкам ОУР Республики Казахстан.")

st.subheader("📂 Загрузка документа")
uploaded_file = st.file_uploader("Выберите файл договора (.txt формат)", type=["txt"])

if st.button("🚀 Запустить команду агентов", type="primary"):
    if not api_key:
        st.error("Ошибка: Пожалуйста, введите ваш секретный API-ключ в левой панели!")
    elif not uploaded_file:
        st.error("Ошибка: Пожалуйста, загрузите текстовый файл договора!")
    else:
        os.environ["OPENAI_API_KEY"] = api_key
        temp_file_path = "uploaded_contract.txt"
        with open(temp_file_path, "wb") as f:
            f.write(uploaded_file.getbuffer())
            
        file_reader = FileReadTool(file_path=temp_file_path)
        
        with st.spinner("🤖 Команда агентов (Бухгалтер, Экономист, Юрист и Адвокат) изучает договор. Пожалуйста, подождите..."):
            accountant = Agent(
                role="Налоговый Бухгалтер ИП на ОУР (Казахстан)",
                goal=f"Рассчитать налоги для ИП на ОУР и зарплатные налоги за {employees_count} сотрудников",
                backstory="Эксперт по налоговому учету РК. Рассчитывает налоги с ФОТ и чистый ИПН от деятельности.",
                tools=[file_reader],
                verbose=True
            )
            economist = Agent(
                role="Главный Экономист",
                goal="Рассчитать точные расходы, чистую прибыль и процент рентабельности контракта",
                backstory="Финансовый аналитик. Использует автоматические вычисления для сведения бюджетов.",
                tools=[file_reader],
                allow_code_execution=True,
                verbose=True
            )
            lawyer = Agent(
                role="Старший Юрист Исполнителя",
                goal="Выявить скрытые ловушки Заказчика, кабальные пункты и защитить права Исполнителя",
                backstory="Эксперт по договорному праву. Проверяет сроки приемки работ и штрафы.",
                tools=[file_reader],
                verbose=True
            )
            litigator = Agent(
                role="Адвокат по арбитражным спорам",
                goal="Сформировать финальный структурированный отчет и Протокол разногласий",
                backstory="Судебный практик, объединяющий расчеты экономики и юридические риски.",
                verbose=True
            )

            task_accounting = Task(
                description=f"Прочитай файл {temp_file_path}. Зафиксируй сумму договора. Рассчитай налоги с ФОТ (составляющего {fot_percent}% от договора) за {employees_count} сотрудников по ставкам РК на ОУР, фиксированные налоги за ИП и ИПН 10% с чистой прибыли.",
                expected_output="Детализация налоговых расходов по договору.",
                agent=accountant
            )
            task_economics = Task(
                description=f"На основе данных от Бухгалтера проведи точный расчет экономики контракта для Исполнителя: Выручка, Расходы на З/П ({fot_percent}%), Сумма налогов, Чистая прибыль и % Рентабельности.",
                expected_output="Таблица математического расчета прибыли и рентабельности контракта.",
                agent=economist
            )
            task_legal = Task(
                description=f"Прочитай файл {temp_file_path}. Изучи разделы об ответственности сторон, порядке сдачи-приемки работ. Найди кабальные формулировки, ущемляющие права ИСПОЛНИТЕЛЯ.",
                expected_output="Юридическое заключение с перечнем опасных для ИСПОЛНИТЕЛЯ формулировок договора.",
                agent=lawyer
            )
            task_final_review = Task(
                description="Объедини финансовые вычисления Экономиста и юридический аудит Юриста. Сформируй итоговый отчет строго в интересах Исполнителя.",
                expected_output="Финальный аналитический отчет для ИСПОЛНИТЕЛЯ:\n1. КОНКРЕТНЫЙ ДЕТАЛЬНЫЙ РАСЧЕТ.\n2. ЮРИДИЧЕСКИЙ БЛОК.\n3. ПРОТОКОЛ РАЗНОГЛАСИЙ в виде четкой таблицы.",
                agent=litigator
            )

            contract_crew = Crew(
                agents=[accountant, economist, lawyer, litigator],
                tasks=[task_accounting, task_economics, task_legal, task_final_review],
                process=Process.sequential
            )
            result = contract_crew.kickoff()
            
        st.success("✅ Анализ договора успешно завершен!")
        st.text_area(label="Отчет ИИ-агентов", value=result.raw, height=600)
        st.download_button(label="📥 Скачать отчет (TXT)", data=result.raw, file_name="audit_report.txt")
        if os.path.exists(temp_file_path):
            os.remove(temp_file_path)
