#include <iostream>
#include <cstring>
#include <vector>
#include <optional>
#include <functional>
#include <algorithm>
#include <memory>
#include <fstream>
#include <iomanip>
#include <chrono>
#include <random>


enum class SortAlgorithms {
    BUBBLE,
    INSERTION,
    MERGE,
    QUICK,
    HEAP,
    STD
};

enum class SortBy {
    NAME,
    COUNTRY,
    POPULATION,
    LAT,
    LON
};

struct SorterSettings {
    std::optional<SortAlgorithms> sorting_algorithms = std::nullopt;
    std::optional<SortBy> sort_by = std::nullopt;
    std::optional<int> num_to_display = std::nullopt;
    bool descending = false;
};

enum class CSVState {
    UnquotedField,
    QuotedField,
    QuotedQuote
};

class csvReader {
public:
    static std::vector<std::vector<std::string> > readCSV(std::istream &in) {
        std::vector<std::vector<std::string> > table;
        std::string row;
        while (!in.eof()) {
            getline(in, row);
            if (in.bad() || in.fail()) {
                break;
            }
            auto fields = readCSVRow(row);
            table.push_back(fields);
        }
        return table;
    }

private:
    static std::vector<std::string> readCSVRow(const std::string &row) {
        CSVState state = CSVState::UnquotedField;
        std::vector<std::string> fields{""};
        size_t i = 0; // index of the current field
        for (char c: row) {
            switch (state) {
                case CSVState::UnquotedField:
                    switch (c) {
                        case ',': // end of field
                            fields.emplace_back("");
                            i++;
                            break;
                        case '"': state = CSVState::QuotedField;
                            break;
                        default: fields[i].push_back(c);
                            break;
                    }
                    break;
                case CSVState::QuotedField:
                    switch (c) {
                        case '"': state = CSVState::QuotedQuote;
                            break;
                        default: fields[i].push_back(c);
                            break;
                    }
                    break;
                case CSVState::QuotedQuote:
                    switch (c) {
                        case ',': // , after closing quote
                            fields.emplace_back("");
                            i++;
                            state = CSVState::UnquotedField;
                            break;
                        case '"': // "" -> "
                            fields[i].push_back('"');
                            state = CSVState::QuotedField;
                            break;
                        default: // end of quote
                            state = CSVState::UnquotedField;
                            break;
                    }
                    break;
            }
        }
        return fields;
    }
};

struct City {
    std::string name, country;
    double lat, lon;
    long population;

    static std::vector<City> get_cities_csv(const std::string &file_path) {
        std::vector<City> cities;
        std::fstream file(file_path);
        if (!file.is_open()) {
            std::cerr << "Error: Could not open file " << file_path << std::endl;
            return cities;
        }
        auto raw_data = csvReader::readCSV(file);


        for (const auto &row: raw_data) {
            try {
                cities.emplace_back(City{
                    row[1],
                    row[4],
                    stod(row[2]),
                    stod(row[3]),
                    stol(row[9]),
                });
                // TODO: silence parsing csv
            } catch (std::invalid_argument const &ex) {
                // std::cerr << "Error: invalid_argument when parsing city csv: " << ex.what() << '\n';
            } catch (std::out_of_range const &ex) {
                // std::cerr << "Error: out_of_range when parsing city csv: " << ex.what() << '\n';
            }
        }

        return cities;
    };
};

std::ostream &operator<<(std::ostream &os, const City &city) {
    os << "City => ";
    os << "Name: " << std::setfill(' ') << std::setw(20) << std::left << city.name << ", ";
    os << "Country: " << std::setfill(' ') << std::setw(20) << std::left << city.country << ", ";
    os << "Lat: " << std::setfill(' ') << std::setw(9) << std::right << std::fixed << std::showpoint <<
            std::setprecision(4) << city.lat << ", ";
    os << "Lon: " << std::setfill(' ') << std::setw(9) << std::right << std::fixed << std::showpoint <<
            std::setprecision(4) << city.lon << ", ";
    os << "Population: " << std::fixed << std::showpoint << std::setprecision(4) << city.population << ", ";
    return os;
}

class CLIParser {
public:
    CLIParser(const int argc, char *argv[]) {
        this->argument_count = argc;
        // just so i can print them
        for (int i = 0; i < argc; i++) {
            this->arguments.emplace_back(argv[i]);
        }

        for (int i = 1; i < argc; i++) {
            if (strstr(argv[i], "-a")) {
                if (i + 1 >= argc) {
                    std::cerr << "Error: Option " << argv[i] << " requires a value" << std::endl;
                } else {
                    std::string str(argv[i + 1]);
                    for (auto &ch: str) {
                        ch = static_cast<char>(tolower(ch));
                    }
                    set_sort_algorithm(str);
                    i++;
                }
                continue;
            }

            if (strstr(argv[i], "-k")) {
                if (i + 1 >= argc) {
                    std::cerr << "Error: Option " << argv[i] << " requires a value" << std::endl;
                } else {
                    std::string str(argv[i + 1]);
                    for (auto &ch: str) {
                        ch = static_cast<char>(tolower(ch));
                    }
                    set_sort_by(str);
                    i++;
                }
                continue;
            }

            if (strstr(argv[i], "-n")) {
                if (i + 1 >= argc) {
                    std::cerr << "Error: Option " << argv[i] << " requires a value" << std::endl;
                } else {
                    try {
                        int temp = std::stoi(argv[i + 1]);
                        this->num_to_display.emplace(temp);
                        i++;
                    } catch (std::invalid_argument const &ex) {
                        std::cerr << "Error: invalid_argument when parsing CLI arguments" << ex.what() << '\n';
                    } catch (std::out_of_range const &ex) {
                        std::cerr << "Error: out_of_range when parsing CLI arguments" << ex.what() << '\n';
                    }
                }
                continue;
            }

            if (strstr(argv[i], "-r")) {
                this->descending = true;
            }
        }
    }

    void print_arguments() const {
        for (int i = 0; i < argument_count; i++) {
            std::cout << arguments[i] << std::endl;
        }
    }

    void print_settings() const {
        std::cout << (this->sort_algorithm.has_value() ? static_cast<int>(this->sort_algorithm.value()) : -1) <<
                std::endl;
        std::cout << (this->sort_by.has_value() ? static_cast<int>(this->sort_by.value()) : -1) << std::endl;
        std::cout << this->num_to_display.value_or(-1) << std::endl;
        std::cout << this->descending << std::endl;
    }

    [[nodiscard]] SorterSettings get_sorter_settings() const {
        return SorterSettings{
            this->sort_algorithm,
            this->sort_by,
            this->num_to_display,
            this->descending
        };
    }

private:
    int argument_count;
    std::vector<std::string> arguments;

    std::optional<SortAlgorithms> sort_algorithm = std::nullopt;
    std::optional<SortBy> sort_by = std::nullopt;
    std::optional<int> num_to_display = std::nullopt;
    bool descending = false;

    void set_sort_by(const std::string &str) {
        if (str == "name") {
            this->sort_by.emplace(SortBy::NAME);
            return;
        }
        if (str == "country") {
            this->sort_by.emplace(SortBy::COUNTRY);
            return;
        }
        if (str == "population") {
            this->sort_by.emplace(SortBy::POPULATION);
            return;
        }
        if (str == "lat") {
            this->sort_by.emplace(SortBy::LAT);
            return;
        }
        if (str == "lon") {
            this->sort_by.emplace(SortBy::LON);
            return;
        }
        std::cerr << "Error: Sort by key " << str << " is not supported" << std::endl;
    }

    void set_sort_algorithm(const std::string &str) {
        if (str == "bubble") {
            this->sort_algorithm.emplace(SortAlgorithms::BUBBLE);
            return;
        }
        if (str == "insertion") {
            this->sort_algorithm.emplace(SortAlgorithms::INSERTION);
            return;
        }
        if (str == "merge") {
            this->sort_algorithm.emplace(SortAlgorithms::MERGE);
            return;
        }
        if (str == "quick") {
            this->sort_algorithm.emplace(SortAlgorithms::QUICK);
            return;
        }
        if (str == "heap") {
            this->sort_algorithm.emplace(SortAlgorithms::HEAP);
            return;
        }
        if (str == "std") {
            this->sort_algorithm.emplace(SortAlgorithms::STD);
            return;
        }
        std::cerr << "Error: Sorting algorithm " << str << " is not supported" << std::endl;
    }
};

template<typename T>
class Sorter {
public:
    virtual ~Sorter() = default;

    Sorter(const std::function<bool(const T &, const T &)> &cmp_fn,
           const SorterSettings &sorter_settings): cmp_fn(cmp_fn),
                                                   sorter_settings(sorter_settings) {
    };

    virtual void sort(std::vector<T> &data) = 0;

    std::chrono::microseconds sort_with_time(std::vector<T> &data) {
        const auto start = std::chrono::steady_clock::now();
        this->sort(data);
        const auto end = std::chrono::steady_clock::now();
        return std::chrono::duration_cast<std::chrono::microseconds>(end - start);
    }

    std::chrono::microseconds benchmark_sort_duration(const std::vector<T> &data, int num_iterations) {
        auto average_duration = std::chrono::microseconds(0);

        std::cout << "\nBenchmark sorting..." << std::endl;
        auto rng = std::default_random_engine{std::random_device{}()};
        for (int i = 0; i < num_iterations; i++) {
            if (i % 10 == 0 || i == num_iterations - 1) {
                int len_num = static_cast<int>(std::to_string(num_iterations - 1).size());
                std::stringstream ss{};
                ss << "\r";
                ss << "Benchmarking (Iteration: ";
                ss << std::setfill(' ') << std::setw(len_num) << std::right << std::to_string(i) << "/";
                ss << std::to_string(num_iterations - 1) << ")";

                std::cout << ss.str();
                std::cout << std::flush;
            }
            std::vector<T> data_copy(data);
            std::shuffle(data_copy.begin(), data_copy.end(), rng);

            const auto start = std::chrono::steady_clock::now();
            this->sort(data_copy);
            const auto end = std::chrono::steady_clock::now();

            average_duration += std::chrono::duration_cast<std::chrono::microseconds>(end - start);
        }
        std::cout << std::endl;

        return average_duration / num_iterations;
    }

    [[nodiscard]] bool correct_sorting(const std::vector<T> &data) const {
        return std::is_sorted(data.begin(), data.end(), this->cmp_fn);
    }

    void print_sorted_data(const std::vector<T> &data) const {
        size_t N = data.size();

        if (this->sorter_settings.num_to_display.has_value()) {
            if (const int num = this->sorter_settings.num_to_display.value(); num >= 0) {
                N = std::min(data.size(), static_cast<size_t>(num));
            }
        }
        for (size_t i = 0; i < N; i++) {
            std::cout << data[i] << std::endl;
        }
    }

protected:
    std::function<bool(const T &, const T &)> cmp_fn;
    SorterSettings sorter_settings;
};

template<typename T>
class BubbleSorter final : public Sorter<T> {
public:
    BubbleSorter(const std::function<bool(const T &, const T &)> &cmp_fn,
                 const SorterSettings &sorter_settings): Sorter<T>(cmp_fn, sorter_settings) {
    };

    // TODO: Bubble Sort Implementation
    void sort(std::vector<T> &data) override {
        const int n = data.size();
        for (int i = 0; i < n - 1; i++) {
            for (int j = 0; j < n - i - 1; j++) {
                if (this->cmp_fn(data[j + 1], data[j])) {
                    std::swap(data[j], data[j + 1]);
                }
            }
        }
    }
};

template<typename T>
class InsertionSorter final : public Sorter<T> {
public:
    InsertionSorter(const std::function<bool(const T &, const T &)> &cmp_fn,
                    const SorterSettings &sorter_settings): Sorter<T>(cmp_fn, sorter_settings) {
    };

    void sort(std::vector<T> &data) override {
        const int n = data.size();
        for (int i = 1; i < n; ++i) {
            auto key = data[i];
            int j = i - 1;

            while (j >= 0 && this->cmp_fn(key, data[j])) {
                data[j + 1] = data[j];
                j = j - 1;
            }
            data[j + 1] = key;
        }
    }
};

template<typename T>
class MergeSorter final : public Sorter<T> {
public:
    MergeSorter(const std::function<bool(const T &, const T &)> &cmp_fn,
                const SorterSettings &sorter_settings): Sorter<T>(cmp_fn, sorter_settings) {
    };

    // TODO: Merge Sort Implementation
    void sort(std::vector<T> &data) override {
    }
};

template<typename T>
class QuickSorter final : public Sorter<T> {
public:
    QuickSorter(const std::function<bool(const T &, const T &)> &cmp_fn,
                const SorterSettings &sorter_settings): Sorter<T>(cmp_fn, sorter_settings) {
    };

    // TODO: Quick Sort Implementation
    void sort(std::vector<T> &data) override {
    }
};

template<typename T>
class HeapSorter final : public Sorter<T> {
public:
    HeapSorter(const std::function<bool(const T &, const T &)> &cmp_fn,
               const SorterSettings &sorter_settings): Sorter<T>(cmp_fn, sorter_settings) {
    };

    // TODO: Heap Sort Implementation
    void sort(std::vector<T> &data) override {
    }
};

template<typename T>
class StdSorter final : public Sorter<T> {
public:
    StdSorter(const std::function<bool(const T &, const T &)> &cmp_fn,
              const SorterSettings &sorter_settings): Sorter<T>(cmp_fn, sorter_settings) {
    };

    void sort(std::vector<T> &data) override {
        std::sort(data.begin(), data.end(), this->cmp_fn);
    }
};

template<typename T>
class SorterFactory {
public:
    explicit SorterFactory(const SorterSettings &sorter_settings): sorter_settings(sorter_settings) {
    };

    [[nodiscard]] std::unique_ptr<Sorter<City> > createSorter() {
        static_assert(sizeof(T) == 0, "NOT IMPLEMENTED");
        return nullptr;
    }

private:
    SorterSettings sorter_settings;
};

template<>
class SorterFactory<City> {
public:
    explicit SorterFactory(const SorterSettings &sorter_settings): sorter_settings(sorter_settings) {
    };

    [[nodiscard]] std::unique_ptr<Sorter<City> > createSorter() {
        std::function<bool(const City &, const City &)> cmp_fn;
        if (!this->sorter_settings.sorting_algorithms.has_value()) {
            std::cerr << "No Sorting Algorithm is provided to SorterFactory" << std::endl;
            return nullptr;
        }
        if (!this->sorter_settings.sort_by.has_value()) {
            std::cerr << "No Sorting Key is provided to SorterFactory" << std::endl;
            return nullptr;
        }
        bool is_descending = this->sorter_settings.descending;
        switch (this->sorter_settings.sort_by.value()) {
            case SortBy::NAME: {
                cmp_fn = [=](const City &a, const City &b) {
                    return is_descending ? (a.name > b.name) : (a.name < b.name);
                };
                break;
            }
            case SortBy::COUNTRY: {
                cmp_fn = [=](const City &a, const City &b) {
                    return is_descending ? (a.country > b.country) : (a.country < b.country);
                };
                break;
            }
            case SortBy::LAT: {
                cmp_fn = [=](const City &a, const City &b) {
                    return is_descending ? (a.lat > b.lat) : (a.lat < b.lat);
                };
                break;
            }
            case SortBy::LON: {
                cmp_fn = [=](const City &a, const City &b) {
                    return is_descending ? (a.lon > b.lon) : (a.lon < b.lon);
                };
                break;
            }
            case SortBy::POPULATION: {
                cmp_fn = [=](const City &a, const City &b) {
                    return is_descending ? (a.population > b.population) : (a.population < b.population);
                };
                break;
            }
        }

        switch (this->sorter_settings.sorting_algorithms.value()) {
            case SortAlgorithms::BUBBLE:
                return std::make_unique<BubbleSorter<City> >(cmp_fn, this->sorter_settings);
            case SortAlgorithms::INSERTION:
                return std::make_unique<InsertionSorter<City> >(cmp_fn, this->sorter_settings);
            case SortAlgorithms::MERGE:
                return std::make_unique<MergeSorter<City> >(cmp_fn, this->sorter_settings);
            case SortAlgorithms::QUICK:
                return std::make_unique<QuickSorter<City> >(cmp_fn, this->sorter_settings);
            case SortAlgorithms::HEAP:
                return std::make_unique<HeapSorter<City> >(cmp_fn, this->sorter_settings);
            case SortAlgorithms::STD:
                return std::make_unique<StdSorter<City> >(cmp_fn, this->sorter_settings);
        }
        return nullptr;
    }

private:
    SorterSettings sorter_settings;
};

int main(const int argc, char *argv[]) {
    auto data = City::get_cities_csv(R"(.\worldcities.csv)");

    const CLIParser parser(argc, argv);
    const SorterSettings sorter_settings = parser.get_sorter_settings();

    SorterFactory<City> sorter_factory(sorter_settings);

    const auto sorter_std = sorter_factory.createSorter();

    if (sorter_std != nullptr) {
        const auto time = sorter_std->benchmark_sort_duration(data, 1000);
        std::cout << "Average Time: " << time.count() << " ms" << std::endl;
        // sorter_std->print_sorted_data(data); not used in benchmark
    }

    return 0;
}
