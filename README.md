import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
import io
from PIL import Image
from IPython.display import display, HTML
from google.colab import files

def load_data_from_string(csv_data):
    """Load data from a CSV string"""
    return pd.read_csv(io.StringIO(csv_data))

def analyze_data(df):
    """Analyze e-waste data and return category counts"""
    category_counts = df['Category'].value_counts()
    return category_counts

def filter_data(df, category=None, min_count=0):
    """Filter data by category and minimum count"""
    filtered = df.copy()
    if category:
        filtered = filtered[filtered['Category'] == category]
    if min_count > 0:
        filtered = filtered[filtered['Count'] >= min_count]
    return filtered

def generate_report(df, output_file='e_waste_report.txt'):
    """Generate comprehensive text report"""
    category_counts = df['Category'].value_counts()

    with open(output_file, 'w') as f:
        f.write("E-Waste Analysis Report\n")
        f.write("="*40 + "\n")
        f.write(f"Total Categories: {len(category_counts)}\n")
        f.write(f"Total Items: {df['Count'].sum()}\n\n")

        f.write("Category Distribution:\n")
        for category, count in category_counts.items():
            f.write(f"- {category}: {count} items\n")

        f.write("\nStatistics:\n")
        stats = df.groupby('Category')['Count'].describe()
        f.write(str(stats))

    print(f"Report saved as '{output_file}'")
    return stats

def create_visualizations(df, output_file='e_waste_visualizations.png'):
    """Create all visualizations in a single figure"""
    plt.figure(figsize=(18, 12))

    # Bar plot
    plt.subplot(2, 2, 1)
    sns.barplot(x='Category', y='Count', data=df, palette='viridis')
    plt.title('E-Waste by Category')
    plt.xticks(rotation=45)

    # Pie chart
    plt.subplot(2, 2, 2)
    df.groupby('Category')['Count'].sum().plot.pie(autopct='%1.1f%%')
    plt.title('Category Distribution')

    # Box plot
    plt.subplot(2, 2, 3)
    sns.boxplot(x='Category', y='Count', data=df)
    plt.title('Distribution by Category')
    plt.xticks(rotation=45)

    # Scatter plot
    plt.subplot(2, 2, 4)
    sns.scatterplot(x='Category', y='Count', data=df, hue='Category', size='Count', sizes=(20, 200))
    plt.title('Quantity Variation')
    plt.xticks(rotation=45)

    plt.tight_layout()
    plt.savefig(output_file)
    plt.close()  # Close the figure to avoid displaying it twice
    
    # Use PIL's Image to open and display
    display(Image.open(output_file))
    print(f"Visualizations saved as '{output_file}'")

def verify_image_colab():
    """Enhanced image verification with better matching"""
    print("\n\nPlease upload an image file (JPEG/PNG) for verification")
    uploaded = files.upload()
    
    if not uploaded:
        print("\n❌ No file was uploaded. Please try again.")
        return None
    
    best_match = None
    best_score = 0
    filename = list(uploaded.keys())[0]
    
    try:
        # Display uploaded image
        img = Image.open(io.BytesIO(uploaded[filename]))
        display(img)
        
        # Simple verification based on filename
        filename_lower = filename.lower()
        categories = {
            'battery': ['batt', 'cell', 'li-ion', 'battery'],
            'cable': ['wire', 'cord', 'lead', 'cable'],
            'electronics': ['phone', 'laptop', 'circuit', 'electronic'],
            'appliance': ['fridge', 'washer', 'oven', 'appliance']
        }
        
        for cat, keywords in categories.items():
            score = sum(key in filename_lower for key in keywords)
            if score > best_score:
                best_score = score
                best_match = cat.capitalize()
        
        if best_match:
            print(f"\n✅ Verification: Identified as {best_match}")
            print(f"Filename analysis score: {best_score}/4")
            print("\nTips: Rename your file with more keywords if unsure")
            return best_match
        else:
            print("\n❓ Verification: Could not confidently identify category")
            print("Valid categories: Batteries, Cables, Electronics, Appliances")
            print("\nPlease rename your file to include one of these keywords:")
            print("- Battery: batt, cell, li-ion, battery")
            print("- Cable: wire, cord, lead, cable")
            print("- Electronics: phone, laptop, circuit, electronic")
            print("- Appliance: fridge, washer, oven, appliance")
            return None
    
    except Exception as e:
        print(f"\n❌ Error processing image: {str(e)}")
        return None

def interactive_menu():
    """Interactive menu for analysis options"""
    print("\nE-Waste Analysis Tool")
    print("="*25)
    print("1. Show all data")
    print("2. Filter by category")
    print("3. Filter by minimum count")
    print("4. Generate report")
    print("5. Visualize data")
    print("6. Upload and verify image")
    print("7. Exit")

    while True:
        try:
            choice = int(input("Enter your choice (1-7): "))
            if 1 <= choice <= 7:
                return choice
            else:
                print("Invalid input. Please enter a number between 1 and 7.")
        except ValueError:
            print("Invalid input. Please enter a number.")

def main():
    try:
        # File upload button styling for Colab
        display(HTML('''
        <style>
        .upload-btn {
            background-color: #4285F4;
            color: white;
            padding: 8px 16px;
            border-radius: 4px;
            border: none;
            cursor: pointer;
        }
        </style>
        '''))
        
        # Your CSV data as a string
        csv_data = """Category,Count
Electronics,100
Batteries,50
Cables,75
Appliances,30"""

        # Load data from string
        df = load_data_from_string(csv_data)
        print("Data loaded successfully from string.")
        display(df)

        while True:
            choice = interactive_menu()

            if choice == 1:
                print("\nAll Data:")
                display(df)

            elif choice == 2:
                print("\nAvailable categories:", ", ".join(df['Category'].unique()))
                category = input("Enter category name: ").strip()
                if category in df['Category'].values:
                    filtered = filter_data(df, category=category)
                    print(f"\nFiltered Data ({category}):")
                    display(filtered)
                else:
                    print(f"\n❌ Invalid category. Please choose from: {', '.join(df['Category'].unique())}")

            elif choice == 3:
                try:
                    min_count = int(input("Enter minimum count: "))
                    if min_count >= 0:
                        filtered = filter_data(df, min_count=min_count)
                        print(f"\nItems with count ≥ {min_count}:")
                        display(filtered)
                    else:
                        print("\n❌ Please enter a positive number")
                except ValueError:
                    print("\n❌ Invalid input. Please enter a number.")

            elif choice == 4:
                output_file = input("Enter output file name for report (default: e_waste_report.txt): ") or 'e_waste_report.txt'
                stats = generate_report(df, output_file=output_file)
                print("\nSummary Statistics:")
                display(stats)

            elif choice == 5:
                output_file = input("Enter output file name for visualizations (default: e_waste_visualizations.png): ") or 'e_waste_visualizations.png'
                create_visualizations(df, output_file=output_file)

            elif choice == 6:
                verified_category = verify_image_colab()
                if verified_category:
                    filtered = filter_data(df, category=verified_category)
                    print(f"\nShowing {verified_category} data:")
                    display(filtered)

            elif choice == 7:
                print("\nThank you for using the E-Waste Analysis Tool!")
                print("Exiting program...")
                break

    except Exception as e:
        print(f"\n❌ An unexpected error occurred: {str(e)}")
    finally:
        print("\nCleanup complete. Have a great day!")

if __name__ == "__main__":
    main()
