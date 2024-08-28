import openpyxl
import pandas as pd
from openpyxl.utils import get_column_letter

def select_sheet(workbook):
    print("사용 가능한 시트:")
    for i, sheet_name in enumerate(workbook.sheetnames, 1):
        print(f"{i}. {sheet_name}")
    
    while True:
        try:
            selection = int(input("사용할 시트 번호를 선택하세요: "))
            if 1 <= selection <= len(workbook.sheetnames):
                return workbook.sheetnames[selection-1]
            else:
                print("올바른 번호를 입력하세요.")
        except ValueError:
            print("숫자를 입력하세요.")

def apply_filter(df):
    print("필터를 적용하려면 다음 형식으로 입력하세요: 컬럼명,연산자,값")
    print("예: Age,>,30 또는 City,==,Seoul")
    print("필터링을 마치려면 빈 줄을 입력하세요.")
    
    while True:
        filter_input = input("필터: ").strip()
        if not filter_input:
            break
        
        try:
            column, operator, value = filter_input.split(',')
            column = column.strip()
            operator = operator.strip()
            value = value.strip()
            
            if operator == '==':
                df = df[df[column] == value]
            elif operator == '>':
                df = df[df[column] > float(value)]
            elif operator == '<':
                df = df[df[column] < float(value)]
            elif operator == '>=':
                df = df[df[column] >= float(value)]
            elif operator == '<=':
                df = df[df[column] <= float(value)]
            elif operator == '!=':
                df = df[df[column] != value]
            else:
                print("지원하지 않는 연산자입니다.")
        except Exception as e:
            print(f"필터 적용 중 오류 발생: {e}")
    
    return df

def select_data(df):
    print("\n필터링된 데이터:")
    print(df)
    
    selected_data = {}
    print("\n복사할 데이터를 선택하세요. 형식: 열이름,행번호")
    print("예: Age,0 (첫 번째 행의 Age 값)")
    print("선택을 마치려면 빈 줄을 입력하세요.")
    
    while True:
        selection = input("선택: ").strip()
        if not selection:
            break
        
        try:
            column, row = selection.split(',')
            column = column.strip()
            row = int(row.strip())
            
            if column in df.columns and 0 <= row < len(df):
                value = df.iloc[row][column]
                selected_data[f"{column}_{row}"] = value
                print(f"선택됨: {column}_{row} = {value}")
            else:
                print("잘못된 열 이름 또는 행 번호입니다.")
        except Exception as e:
            print(f"데이터 선택 중 오류 발생: {e}")
    
    return selected_data

def copy_data(selected_data, target_sheet, mappings):
    for target_cell, source_key in mappings.items():
        if source_key in selected_data:
            target_sheet[target_cell] = selected_data[source_key]
        else:
            print(f"경고: '{source_key}' 데이터를 찾을 수 없습니다.")

def main():
    # RawData 파일 열기
    raw_data_path = input("RawData 엑셀 파일 경로를 입력하세요: ")
    raw_workbook = openpyxl.load_workbook(raw_data_path)
    
    # 시트 선택
    selected_sheet = select_sheet(raw_workbook)
    
    # pandas DataFrame으로 변환
    df = pd.read_excel(raw_data_path, sheet_name=selected_sheet)
    
    # 필터 적용
    df = apply_filter(df)
    
    # 데이터 선택
    selected_data = select_data(df)
    
    # 요약 레포트 파일 생성 또는 열기
    report_path = input("요약 레포트 엑셀 파일 경로를 입력하세요 (없으면 새로 생성됩니다): ")
    try:
        report_workbook = openpyxl.load_workbook(report_path)
    except FileNotFoundError:
        report_workbook = openpyxl.Workbook()
    
    report_sheet = report_workbook.active
    
    # 데이터 매핑 정의
    mappings = {}
    print("데이터 매핑을 입력하세요. 완료하려면 빈 줄을 입력하세요.")
    print("형식: 대상 셀, 선택한 데이터 키 (예: A1, Age_0)")
    while True:
        mapping = input("매핑: ").strip()
        if not mapping:
            break
        target, source = map(str.strip, mapping.split(','))
        mappings[target] = source
    
    # 데이터 복사
    copy_data(selected_data, report_sheet, mappings)
    
    # 요약 레포트 저장
    report_workbook.save(report_path)
    print(f"요약 레포트가 {report_path}에 저장되었습니다.")

if __name__ == "__main__":
    main()
